## Import
Foram importadas as bibliotecas:

*FastAPI: Framework para construir APIs.
*requests: Biblioteca para fazer requisições HTTP.
*re*: Biblioteca para expressões regulares.
*typing: Tipagem para listas, dicionários e união de tipos.
*pydantic: Biblioteca para validação de dados.
*datetime: Biblioteca para manipulação de datas.

## Sobre o Back-end

O back-end possui vários algoritmos de raspagem de dados onde eles recebem um dicionário como entrada e em seguida acessam 
a api do querido diário e retornam listas de dicionários contendo os dados obtidos do diário oficial da união através da api do querido diário. 

 
## Puxador_compras

Este algoritmo recebe de entrada dois dados em um dicionário, o `published_since: date` como data e inicio da busca e com o `date` sendo o formato da hora no padrão americano e o `published_until: date` sendo a data para o fim da busca.

ficando neste formato:

```python
{
published_since: date,
published_until: date
}
```

O retorno deste algoritmo é uma lista de dicionarios com os seguintes dados encontrados:

```python
({
"data": data,
"empresa": empresa,
"cnpj": cnpj,
"objeto": objeto,
"valor": valor_float
})
```



## Como rodar o Back-end


<h1>Como utilizar o WebScrapper com a API do Querido diário</h1>


Os WebScrapper são progamas em python que filtram os dados recebidos através da API do Querido diário



### Requisitos:
+ Python 3.10 ou superior
+ Requests
+ Re

A API pode ser acessada por este [link](https://queridodiario.ok.org.br/api/docs#/)

### Configurando a API
Primeiro é nescessario ditar os parâmetros na opção de gazetas:
+ ID do município que deseja realizar a busca
+ Intervalo de tempo em que deseja buscar
+ String que deseja
+ o numero de caractéres que irá retornar
+ Numero maximo de resultados
+ Numero maximo de resultaods por gazeta retornada
+ Ordenar a Data
![image](https://github.com/user-attachments/assets/74b48bc4-5ba7-49fd-97c3-1593f1bf837f)


Ao executar a APi ela retornará o seguinte link
![image](https://github.com/user-attachments/assets/ebde8164-2ee5-4fb0-8aa6-14b82f64397b)

Este link retorna um ou varios Json de acordo com os parâmetros buscados
Temos:
+ O total de gazetas
+ Id do territorio
+ Data
+ Data em que foi processado
+ URL do diario em PDF
+ Nome do territorio
+ Sigla do estado
+ A string que foi buscada dentro de um excerpt
+ verifica se o diario é uma versão extra-oficial
+ link para o diario oficial em txt
![image](https://github.com/user-attachments/assets/bfbb22ea-9022-4aa9-a624-abbf30eeeffc)


### Utilizando o progama em python

usarei de exemplo o codigo do puxador de verbas de escola

Como parametro de query coloquei "Nº CRE/UE Capital Total" pois é assim que aparecem as verbas de escola no arquivo .txt
![image](https://github.com/user-attachments/assets/517990ca-08ec-49fb-a65a-836e708e3e53)


Utilizando a biblioteca Requests, colocando o link com o comando Request.get(link)
~~~Python
# Fazendo a solicitação GET para a API
response = requests.get(
    F'https://queridodiario.ok.org.br/api/gazettes?territory_ids=5300108&published_since=2023-06-26&published_until=2023-06-30&querystring=%22RECONHECIMENTO%20DE%20D%C3%8DVIDA%22&excerpt_size=500&number_of_excerpts=100000&pre_tags=&post_tags=&size=10000&sort_by=descending_date'
)

~~~

Apos isso o resultado é tratado e reorganizado para que valores moneterios utilizem "." ao invez de "," para que sejam tratados em python

~~~Python
# Verificando se a solicitação foi bem-sucedida
if response.status_code == 200:
    # Convertendo a resposta para JSON
    dados = response.json()

    # Acessando a lista de gazettes dentro dos dados
    gazettes = dados['gazettes']

    # Conjunto para armazenar as datas já vistas
    datas_vistas = set()

    # Iterando sobre cada gazette na lista
    for gazette in gazettes:
        # Acessando a data de cada gazette
         data = gazette.get('date')

        # Variável para armazenar o total gasto no dia
        total_gasto_dia = 0.0

         # Verifica se a data está presente e não é uma string vazia
    if data and data not in datas_vistas:
        # Adiciona a data ao conjunto de datas vistas
        datas_vistas.add(data)

        # Obtém a lista de excertos de cada gazette
        excertos = gazette.get('excerpts', [])
        total_diario = 0

         for excerto in excertos:
                # Removendo quebras de linha e hifens no meio das palavras
                excerto = re.sub(r'(\w)-\n(\w)', r'\1\2', excerto)
                excerto = excerto.replace('\n', ' ')
                # Usando expressões regulares para extrair informações da empresa, objeto e valor
                empresa_match = re.search(r'empresa\s+([\w\s\-ÇçÉéÁáÍíÓóÚúÃãÕõâêîôûÂÊÎÔÛäëïöüÄËÏÖÜ]+)\s+-\s+CNPJ:\s+(\d{2}\.\d{3}\.\d{3}/\d{4}-\d{2})', excerto, re.DOTALL)
                objeto_match = re.search(r'cujo objeto é a aquisição do item identificado pelo Código SES\s+\d+\s+-\s+([^\n,]+)', excerto, re.DOTALL)
                valor_match = re.search(r'valor global de R\$ ([\d,.]+)', excerto)
                if empresa_match and objeto_match and valor_match:
                    # Extraindo e limpando as informações encontradas
                    empresa = empresa_match.group(1).strip()
                    cnpj = empresa_match.group(2).strip()
                    objeto = objeto_match.group(1).strip()
                    objeto = re.sub(r',\s*para atender as necessidades.*', '', objeto)
                    valor = valor_match.group(1)
                    valor_limpo = re.sub(r'[^\d,.]', '', valor)
                    valor_limpo = valor_limpo.replace('.', '')
                    valor_limpo = valor_limpo.replace(',', '.')
                    valor_float = float(valor_limpo)
                    total_diario += valor_float
                    # Adicionando as informações extraídas à lista de resultados
                    results.append({
                        "data": data,
                        "empresa": empresa,
                        "cnpj": cnpj,
                        "objeto": objeto,
                        "valor": valor_float
                    })
        # Retornando a lista de resultados
        return results
    else:
        # Se a solicitação falhar, retorna o código de status
        return response.status_code
                        