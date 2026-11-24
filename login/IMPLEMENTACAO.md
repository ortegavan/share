# Login novo no Cactus (Angular)

O mínimo para levar este protótipo para um componente Angular. O protótipo é a referência: `index.html`, `style.css`, `css/animation.css` e `css/login.css`.

## 1. Arquivos

| Do protótipo | Para o Cactus |
|---|---|
| `img/bg.jpg`, `img/icone.svg`, `img/logo-branco.svg` | `src/assets/v2/img/` |
| Conteúdo do `<body>` do `index.html` (`<main class="login">` + `<div class="abertura">`) | `login.component.html` |
| `style.css` + `css/animation.css` + `css/login.css`, **nessa ordem** | `login.component.css` (um arquivo só) |

O `<script>` do `<head>` **não** vai para o `index.html` do Angular. A lógica do cookie fica no componente (passo 3).

Se o Cactus ainda não carrega a fonte Poppins (400, 500 e 700), adicione o `<link>` do Google Fonts no `src/index.html`.

## 2. Ajustes no CSS

Com o encapsulamento padrão do Angular, `:root`, `html` e `body` não funcionam dentro do CSS de componente, e afetariam o app inteiro se fossem globais. Troque tudo pelo `:host`:

**a) Variáveis**: todo `:root {` vira `:host {`, inclusive o que está dentro do `@media (max-width: 720px)`.

**b) Página**: o bloco `html, body { ... }` e o `body { ... }` de `style.css` saem e viram:

```css
:host {
    position: fixed;
    inset: 0;
    display: block;
    overflow: hidden;
    background: var(--cor-login);
    font-family: "Poppins", system-ui, sans-serif;
}
```

**c) Visita repetida** (fim de `animation.css`):

```css
:host(.abertura-vista) {
    --cortina-inicio: 0.3s;
}

:host(.abertura-vista) .marca__simbolo,
:host(.abertura-vista) .marca__quadrado,
:host(.abertura-vista) .marca__letra {
    animation: none;
}

:host(.abertura-vista) .marca__palavra {
    grid-template-columns: 1fr;
    animation: none;
}
```

**d) Imagem de fundo**: em `.login::before`, troque `url("../img/bg.jpg")` por `url("/assets/v2/img/bg.jpg")`.

**e) Template**: `src="img/..."` vira `src="assets/v2/img/..."` (ícone e logo branco).

O resto do CSS vai como está: nomes de classe, keyframes, `prefers-reduced-motion`.

## 3. Componente: cookie da abertura

```ts
import { Component, inject } from '@angular/core';
import { DOCUMENT } from '@angular/common';

// Trocar a versão faz todo mundo ver a abertura completa de novo
const ABERTURA = { nome: 'cactus_abertura', versao: 'v1', dias: 20 };

@Component({
    selector: 'app-login',
    templateUrl: './login.component.html',
    styleUrl: './login.component.css',
    host: { '[class.abertura-vista]': 'aberturaVista' },
})
export class LoginComponent {
    private readonly document = inject(DOCUMENT);

    // Primeira visita: abertura completa e grava o cookie.
    // Próximas (até o cookie expirar): logo pronto e cortina logo em seguida.
    // O cookie não é renovado a cada visita.
    readonly aberturaVista = this.verificarAbertura();

    private verificarAbertura(): boolean {
        const { nome, versao, dias } = ABERTURA;
        const cookie = `${nome}=${versao}`;

        if (this.document.cookie.split('; ').includes(cookie)) {
            return true;
        }

        this.document.cookie = `${cookie}; max-age=${dias * 24 * 60 * 60}; path=/; SameSite=Lax`;
        return false;
    }
}
```

A classe é decidida antes da primeira renderização do componente, então a página não pisca. Como a verificação roda toda vez que o componente é criado, ela também vale quando se volta ao login pelo router (logout sem recarregar a página).

## 4. Formulário

O HTML do cartão é só a estrutura visual. Ligue nele o formulário que o Cactus já usa (`formGroup`, `formControlName`, `(ngSubmit)`, mensagens de erro), mantendo as classes `campo`, `campo__rotulo`, `campo__input` e `botao`.

## 5. Para conferir

- **CSS global do app**: resets, Angular Material ou estilos em `button`/`input` podem sobrescrever os campos e botões do cartão.
- **`style="--i: 0"`** no template: se o Angular não aplicar a variável, use `[style.--i]="0"`.
- **SSR**: se o Cactus renderiza no servidor, o `document.cookie` lá vem vazio. Nesse caso, leia o cookie da requisição, ou rode a verificação só no navegador (`isPlatformBrowser`).
- **Movimento reduzido**: com `prefers-reduced-motion` ativo, a abertura não aparece e a login entra com um fade curto, independentemente do cookie.
- **Para rever a abertura durante os testes**: `document.cookie = "cactus_abertura=; max-age=0; path=/"` no console.
