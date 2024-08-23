# Api

## Sobre a Api

A API está dividida em módulos, onde cada pasta é um módulo com sua própria rota e particularidade de acesso, com o arquivo main.py orquestrando isso. A sua construção é feita em Python e FastAPI.

## Como rodar o ambiente da API

### Algumas informações

A api e feita em `python` usando `fastapi` e esta rodando em um ambiente virtual python com suas dependências ja instaladas.

### Pré-requisitos

Ter o Python 3.12.3 ou superior instalado.

Ter o pip 24.0 ou superior instalado.

### No Windows

Tentei. Não roda, ativa o WSL ou instala um ubuntu e segue os passos do linux 👍

### No linux (ubuntu)

Dentro da pasta raiz do projeto `Gastos-DF-Squad12` execulte os seguintes comandos:

Para ativar o ambiente virtual do python execute:
```
source venv/bin/activate
```

Em seguida para abrir um servidor local e visualizar no seu navegador execute:
```
python3 api/main.py
```

Será aberto no endereço <a href="http://localhost:8001/">http://localhost:8001/</a>.

O seu docs estará no endereço <a href="http://localhost:8001/docs">http://localhost:8001/docs</a>.

Mais informações sobre o ambiente virtal python podem ser encontrado em <a href="https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/">https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/</a>.

