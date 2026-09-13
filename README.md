# Gestão de Mudanças — versão GitHub Pages + Firestore

Este pacote contém a versão revisada do app **Gestão de Mudanças**, adaptada
para rodar publicada no **GitHub Pages** com os dados salvos no
**Firebase Firestore**, em vez de `localStorage` do navegador.

Nenhuma funcionalidade do app foi alterada: os mesmos planos, grupos de
trabalho, etapas (Traduzir → Localizar → Atribuir → Validar → Agir), plano
consolidado, impressão e exportação/importação de JSON continuam
funcionando exatamente como antes. O que mudou foi **onde os dados ficam
guardados** (agora na nuvem, não só no navegador de quem usa) e **de onde
vem a logo** (agora de um arquivo de imagem, não mais embutida no código).

## O que foi revisado

- **Persistência**: `localStorage` → documento único no Firestore
  (coleção `gestaoMudancas`, documento `estado`). O app carrega esse
  documento ao abrir e grava nele a cada alteração, do mesmo jeito que
  antes gravava no navegador — só que agora fica acessível de qualquer
  dispositivo, pelo mesmo link.
- **Exemplo removido do código**: o plano de exemplo que vinha embutido
  no HTML foi retirado do arquivo e exportado para
  `exemplo-gestao-mudancas.json`. Ele não é mais carregado
  automaticamente — o app abre vazio (convidando a criar o primeiro
  plano) e o exemplo só entra no banco se você importar esse arquivo
  pelo próprio botão **"Importar JSON"** que já existe na tela inicial
  (isso já grava direto no Firestore).
- **Logo alterável**: as duas logos (Pernambuco na Reforma e Fundação
  Dom Cabral), que antes estavam coladas no HTML como texto
  base64 (o que deixava o arquivo gigante e a logo fixa no código),
  agora são carregadas de dois arquivos de imagem separados, dentro da
  pasta `assets/`. Basta substituir esses arquivos no repositório do
  GitHub (mantendo o mesmo nome) para trocar a logo, sem tocar no
  código do app.
- Arquivo ficou ~97% mais leve (as imagens deixaram de estar duplicadas
  em texto dentro do HTML).

## Estrutura dos arquivos

```
gestao-de-mudancas/
├── index.html                     ← o app (abrir/publicar este arquivo)
├── assets/
│   ├── logo-esquerdo.png          ← logo "Pernambuco na Reforma"
│   └── logo-direita.png           ← logo "Fundação Dom Cabral"
├── exemplo-gestao-mudancas.json   ← plano de exemplo (opcional, para importar)
├── firestore.rules                ← regras de segurança do Firestore
├── .gitignore
└── README.md                      ← este arquivo
```

## Passo a passo da implantação

### 1. Confirmar o projeto no Firebase

O app já está configurado para usar o projeto Firebase abaixo (os dados
já estão embutidos em `index.html`, não precisa editar nada):

- Project ID: `gestao-mudancas`
- Se esse projeto ainda não existir, crie-o em
  [console.firebase.google.com](https://console.firebase.google.com/) →
  **Adicionar projeto** → nome `gestao-mudancas` (ou o nome que já tiver
  sido reservado com esse Project ID).

### 2. Ativar o Firestore

1. No console do Firebase, abra **Build → Firestore Database**.
2. Clique em **Criar banco de dados**.
3. Escolha o modo **produção** e a localização mais próxima (ex.:
   `southamerica-east1`).
4. Em **Regras**, cole o conteúdo do arquivo `firestore.rules` deste
   pacote e clique em **Publicar**.

### 3. Criar o repositório no GitHub

1. Crie um repositório novo (pode ser público ou privado — GitHub Pages
   funciona nos dois casos em contas com esse recurso liberado).
2. Envie todo o conteúdo desta pasta para a raiz do repositório
   (`index.html`, a pasta `assets/`, etc.), mantendo os nomes e a
   estrutura de pastas exatamente como estão.

```bash
git init
git add .
git commit -m "Gestão de Mudanças — versão Firestore + GitHub Pages"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/SEU_REPOSITORIO.git
git push -u origin main
```

### 4. Ativar o GitHub Pages

1. No repositório, vá em **Settings → Pages**.
2. Em **Source**, selecione a branch `main` e a pasta `/ (root)`.
3. Salve. Em alguns minutos o GitHub mostrará o link público, algo como
   `https://SEU_USUARIO.github.io/SEU_REPOSITORIO/`.

### 5. Testar

1. Abra o link do GitHub Pages.
2. O app deve mostrar "Carregando dados..." por um instante e depois abrir
   vazio, pronto para o primeiro plano.
3. Crie um plano de teste, atualize a página e confirme que os dados
   continuam lá (prova de que está salvando no Firestore, não só na
   memória do navegador).

### 6. (Opcional) Carregar o plano de exemplo

Se quiser começar com o exemplo pronto em vez de um app vazio:

1. Abra o app publicado.
2. Clique em **"Importar JSON"** na tela inicial.
3. Selecione o arquivo `exemplo-gestao-mudancas.json` deste pacote.
4. Confirme a importação — isso grava o plano de exemplo no Firestore.

### 7. Trocar a logo quando precisar

Para trocar qualquer uma das duas logos no futuro, sem mexer no código:

1. Substitua o arquivo correspondente dentro de `assets/`
   (`logo-esquerdo.png` ou `logo-direita.png`) por uma imagem nova,
   **mantendo o mesmo nome de arquivo**.
2. Suba a alteração para o GitHub (`git add`, `git commit`, `git push`).
3. O GitHub Pages atualiza automaticamente em alguns minutos.

Se preferir usar nomes de arquivo diferentes ou formatos diferentes
(ex. `.svg`), basta editar as duas linhas `<img src="assets/...">` no
`index.html` (aparecem duas vezes cada: uma no cabeçalho, outra no
modelo de impressão).

## Observações técnicas

- O app continua sendo um único arquivo HTML/CSS/JS (sem build, sem
  dependências além do SDK do Firebase, carregado via CDN).
- Os dados de todos os planos ficam em **um único documento** do
  Firestore, do mesmo jeito que antes ficavam em um único item do
  `localStorage`. Isso preserva o comportamento original (sem
  colaboração simultânea em tempo real entre abas/dispositivos — quem
  salvar por último sobrescreve). Se no futuro for necessário suportar
  várias pessoas editando ao mesmo tempo, o próximo passo natural seria
  separar os planos em subcoleções do Firestore — isso é uma mudança de
  arquitetura maior e não foi feita aqui para não alterar o
  funcionamento atual do app.
- A regra do Firestore incluída libera leitura/escrita para quem tiver
  o link do app (sem login), reproduzindo o mesmo nível de "proteção"
  que o `localStorage` já tinha (nenhum). Veja o comentário dentro de
  `firestore.rules` para uma sugestão de evolução futura com autenticação.
