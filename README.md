# Rita & Tiago — Bíblia de Casamento (app)

App em forma de PWA (Progressive Web App) com o orçamento, os convidados,
o controlo de gastos e as mesas do casamento. Instala-se no iPhone a partir
do Safari, sem passar pela App Store.

## Ficheiros

- `index.html` — a app em si (ponto de entrada, é o que abre no telemóvel)
- `manifest.json` — diz ao telemóvel o nome, o ícone e as cores da app
- `sw.js` — service worker, permite abrir a app mesmo sem internet
- `icon-180.png`, `icon-192.png`, `icon-512.png` — ícone da app em vários tamanhos
- `prototipo-iphone.html` — versão só para veres num browser de computador,
  dentro de uma moldura de iPhone (não é preciso para a instalação, é só para preview)

## Passo 1 — Criar o repositório no GitHub

1. Entra em [github.com](https://github.com) (cria conta se ainda não tiveres).
2. Clica no botão verde **"New"** (ou o "+" no canto superior direito → **"New repository"**).
3. Dá-lhe um nome, por exemplo `rita-tiago-app`.
4. Deixa-o **Public** (tem de ser público para o GitHub Pages funcionar de graça).
5. Não marques nenhuma opção extra ("Add a README", etc.) — vamos enviar os nossos próprios ficheiros.
6. Clica **"Create repository"**.

## Passo 2 — Enviar os ficheiros (sem usar o terminal)

1. Na página do repositório que acabaste de criar, clica em **"uploading an existing file"**
   (ou "Add file" → "Upload files").
2. Arrasta **todos os ficheiros desta pasta** (incluindo os `.png` e o `manifest.json`,
   não só o `index.html`) para a janela do browser.
3. Em baixo, escreve uma mensagem tipo `primeira versão da app` e clica **"Commit changes"**.

## Passo 3 — Ativar o GitHub Pages

1. No repositório, vai a **Settings** (menu de cima).
2. No menu da esquerda, clica em **Pages**.
3. Em **"Build and deployment" → "Branch"**, escolhe `main` e a pasta `/ (root)`.
4. Clica **Save**.
5. Espera cerca de 1 minuto e atualiza a página — vai aparecer um link tipo:
   `https://o-teu-utilizador.github.io/rita-tiago-app/`

Esse é o endereço da tua app, já em HTTPS (obrigatório para funcionar bem como PWA).

## Passo 4 — Instalar no iPhone

1. Abre esse link no **Safari** do iPhone (tem de ser Safari, o Chrome não permite instalar).
2. Toca no ícone de **Partilhar** (o quadrado com a seta a apontar para cima).
3. Desliza para baixo e toca em **"Adicionar ao Ecrã Principal"**.
4. Toca em **"Adicionar"** no canto superior direito.

Vais ter um ícone "R & T" no ecrã principal do teu iPhone que abre a app em ecrã
inteiro, sem barra de endereço — tal como uma app normal.

## Atualizar a app mais tarde

Sempre que quiseres alterar alguma coisa, basta voltar ao repositório no GitHub,
substituir o(s) ficheiro(s) (o mesmo botão "Add file" → "Upload files", com o
mesmo nome, substitui o antigo) e esperar cerca de 1 minuto. Quem já tiver a app
instalada no ecrã principal recebe a versão nova automaticamente da próxima vez
que a abrir (com internet).

## Novidade — login e dados partilhados em tempo real

A app já tem um ecrã de login (só entra quem tu autorizares, por email) e os
dados (convidados, gastos, mesas) passam a ficar guardados **num só sítio
partilhado**, não mais no telemóvel de cada pessoa — quando tu marcas um
convidado como confirmado, o Tiago vê isso aparecer no telemóvel dele quase
na hora, e vice-versa.

Isto usa o **Firebase** (serviço gratuito da Google para este tipo de apps).
Precisas de fazer esta configuração uma única vez.

### A — Criar o projeto Firebase

1. Entra em [console.firebase.google.com](https://console.firebase.google.com)
   com uma conta Google.
2. **"Add project"** (ou "Criar projeto") → dá-lhe um nome, ex: `rita-tiago-app`.
3. Podes desligar o Google Analytics (não é preciso) → **"Create project"**.

### B — Registar a app e copiar a configuração

1. Já dentro do projeto, clica no ícone **`</>`** ("Web") para adicionar uma app web.
2. Dá-lhe um nome (ex: `rita-tiago-app`) → **"Register app"**.
3. Vai aparecer um bloco de código com `const firebaseConfig = { apiKey: ..., ... }`.
   **Copia esse objeto todo.**
4. Abre o ficheiro `index.html` (no teu computador, ou diretamente no GitHub
   com o ícone de lápis) e substitui este bloco, perto do início do `<script>`:

   ```js
   const firebaseConfig = {
     apiKey: "COLOCA_AQUI_A_TUA_API_KEY",
     authDomain: "COLOCA_AQUI.firebaseapp.com",
     projectId: "COLOCA_AQUI_O_PROJECT_ID",
     storageBucket: "COLOCA_AQUI.appspot.com",
     messagingSenderId: "COLOCA_AQUI",
     appId: "COLOCA_AQUI"
   };
   ```

   pelo que copiaste no passo 3 (os valores reais do teu projeto).

### C — Ativar o login por email (sem password)

1. No menu da esquerda da consola Firebase: **Authentication** → **"Get started"**.
2. Na aba **"Sign-in method"**, clica em **"Email/Password"**.
3. Ativa as duas opções: **"Email/Password"** e, logo abaixo, **"Email link
   (passwordless sign-in)"**. Guarda.
4. Ainda em Authentication, vai a **Settings → Authorized domains** e
   adiciona o teu domínio do GitHub Pages, por exemplo:
   `o-teu-utilizador.github.io` (sem `https://` e sem a parte final do link).

### D — Criar a base de dados (Firestore)

1. No menu da esquerda: **Firestore Database** → **"Create database"**.
2. Escolhe **"Start in production mode"** → escolhe uma região perto de
   Portugal (ex: `eur3 (europe-west)`) → **"Enable"**.
3. Vai à aba **"Rules"** (regras) dentro do Firestore e substitui tudo por:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /weddings/rita-tiago {
         allow read, write: if request.auth != null &&
           request.auth.token.email in [
             'rita@exemplo.com',
             'tiago@exemplo.com'
           ];
       }
     }
   }
   ```

4. Troca os emails de exemplo pelos emails que queres autorizar → **"Publish"**.

   **É aqui que decides quem tem acesso.** Para autorizar ou remover alguém
   mais tarde, volta a esta aba, edita a lista de emails e clica "Publish" —
   não precisas de mexer no GitHub para isso.

### E — Publicar as alterações

1. Guarda o `index.html` (já com a tua configuração real, do passo B) e
   sobe-o para o GitHub (Passo "Atualizar a app" acima).
2. Espera ~1 minuto pelo GitHub Pages e abre o link da app.
3. Escreve um dos emails autorizados → **"Enviar link de acesso"**.
4. Vai ao email (verifica também o spam) e abre o link — a app entra
   automaticamente e mostra os dados partilhados.

### Como funciona no dia a dia

- Só quem tiver o email na lista das regras (passo D) consegue entrar.
- Cada pessoa autorizada recebe sempre um link novo por email quando quer
  entrar — não há password para esquecer ou partilhar.
- Qualquer alteração feita por uma pessoa (marcar convidado, registar gasto,
  atribuir mesa) aparece nos outros telemóveis em poucos segundos, desde
  que tenham internet.
- Sem internet, a app continua a abrir (graças ao `sw.js`), mas não é
  possível entrar nem sincronizar até voltar a haver ligação.

### Nota sobre o `prototipo-iphone.html`

Esse ficheiro fica como estava — sem login, com dados de exemplo fixos —
só serve para veres rapidamente o design num computador. A versão real,
com login e dados partilhados, é sempre o `index.html`.
