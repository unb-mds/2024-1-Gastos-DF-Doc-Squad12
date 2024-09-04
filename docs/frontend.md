# Front End

Nosso front end foi desenvolvido utilizando HTML, CSS e JavaScript. Ele recebe os dados raspados em json e os exibe em formato de gráficos.

Está organizado da seguinte maneira: todos os JSONs usados estão na pasta WEB, e todas as páginas HTML também estão presentes na mesma pasta. Há uma pasta imagens para separar as imagens usadas e uma pasta CSS para separar os scripts CSS utilizados. Além disso, há também a pasta Bases web, onde está a lógica do projeto.

## Como rodar o Projeto

Para roda o Projeto localmente pela linha de comando, siga os passos abaixo:

1. **Abra o Terminal ou Prompt de Comando:**
   - No Windows, você pode usar o Prompt de Comando ou PowerShell.
   - No macOS ou Linux, use o Terminal.

2. **Navegue até o Diretório do Projeto:**
   Use o comando `cd` para mudar para o diretório onde os arquivos HTML estão localizados. Por exemplo:
   ```bash
   cd /caminho/para/seu/projeto
   
3. Inicie um Servidor Web Local:
Você pode usar uma ferramenta como http-server para iniciar um servidor local. Se não tiver http-server instalado, você pode instalá-lo via npm (Node Package Manager). Primeiro, instale o Node.js e npm, se ainda não o fez, e depois execute:
  ```bash
  npm install -g http-server
   ```
  Após a instalação, inicie o servidor com:
  ```
  http-server
  ```
Acesse o Projeto no Navegador:
Após iniciar o servidor, você verá uma URL local no terminal, como http://127.0.0.1:8080. Abra essa URL no seu navegador para ver o Projeto em funcionamento.

Verifique os Resultados:
Navegue pelas páginas e funcionalidades para garantir que tudo esteja funcionando conforme o esperado.
