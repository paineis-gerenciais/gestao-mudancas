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

- **Persistência com acesso simultâneo em tempo real**: cada plano agora
  é gravado como um documento próprio (`plans/{id}`), e cada grupo de
  trabalho como um documento dentro dele
  (`plans/{id}/groups/{id}`) — em vez de tudo junto num único
  documento como antes. Isso permite que várias pessoas usem o app ao
  mesmo tempo, cada uma vendo as alterações das outras aparecerem
  automaticamente na tela, sem precisar recarregar a página — inclusive
  duas pessoas em grupos diferentes do mesmo plano, editando em
  paralelo sem uma atrapalhar a outra. Mais detalhes em
  "Acesso simultâneo", abaixo.
- **Migração automática dos dados antigos**: se o Firestore já tinha
  dados no formato anterior (documento único), a primeira vez que o app
  for aberto após este deploy ele migra tudo automaticamente para a
  nova estrutura, sem precisar de nenhuma ação manual.
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

## Atualizando uma implantação já existente

Se este app já estava publicado (com a versão de documento único), a
atualização é simples:

1. Publique as novas `firestore.rules` no console do Firebase (elas
   mudaram — veja o passo 2 abaixo).
2. Suba o novo `index.html` para o GitHub (substitua o arquivo, mesmo
   nome).
3. Pronto. Na primeira vez que alguém abrir o app depois disso, a
   migração automática copia os dados antigos para a nova estrutura —
   não precisa fazer nada manualmente no Firestore.

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

## Acesso simultâneo

- Cada **plano** é um documento (`plans/{planId}`) e cada **grupo de
  trabalho** é outro documento dentro dele
  (`plans/{planId}/groups/{groupId}`). O app mantém dois "ouvintes" em
  tempo real do Firestore (um para a lista de planos, outro para todos
  os grupos de todos os planos) — qualquer alteração salva por
  qualquer pessoa chega automaticamente para todas as outras que
  estiverem com o app aberto, sem precisar recarregar.
- **Duas pessoas em planos ou grupos diferentes**: trabalham em
  paralelo sem qualquer risco de uma sobrescrever os dados da outra,
  porque cada uma grava num documento distinto.
- **Duas pessoas no mesmo grupo ao mesmo tempo**: cada uma vê os fatos,
  impactos, responsabilidades, validações e ações que a outra for
  salvando aparecerem na tela. Se as duas editarem exatamente o mesmo
  item quase ao mesmo tempo, **a última gravação vence** (a mesma regra
  simples que já existia antes, com `localStorage` — sem bloqueio, sem
  aviso de conflito).
- **Enquanto uma janela de edição (modal) está aberta**, o app pausa a
  atualização da tela para não fechar o que você está digitando ou
  perder o foco do campo — mas continua recebendo os dados por trás.
  Assim que você salva ou fecha a janela, a tela é atualizada com tudo
  o que chegou nesse meio-tempo.
- Não há tela de login: qualquer pessoa com o link do app publicado
  consegue ler e editar os dados (mesmo nível de "proteção" que o
  `localStorage` já tinha antes — nenhum). Veja o comentário dentro de
  `firestore.rules` para uma sugestão de evolução futura com
  autenticação.

## Observações técnicas

- O app continua sendo um único arquivo HTML/CSS/JS (sem build, sem
  dependências além do SDK do Firebase, carregado via CDN).
- A exportação/importação de JSON (botões "Exportar" e "Importar JSON")
  continua no mesmo formato de antes — um backup exportado da versão
  anterior pode ser importado normalmente nesta versão, e vice-versa.
  Importar um arquivo agora substitui os dados **para todas as pessoas
  conectadas**, não só no seu navegador.
- O documento antigo (`gestaoMudancas/estado`) só é lido uma vez, na
  migração automática — depois disso o app não usa mais essa coleção.
  Pode apagá-la do Firestore mais adiante, se quiser (não é obrigatório).
