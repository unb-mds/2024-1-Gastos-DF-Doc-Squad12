# Documentação do Backend - Projeto Querido Diário

## Visão Geral

Este backend consiste em cinco algoritmos que interagem com a API "Querido Diário". Cada algoritmo é responsável por realizar uma busca específica na API, filtrar os dados recebidos e armazená-los em um arquivo JSON. Esses arquivos JSON são atualizados automaticamente por um workflow do GitHub, garantindo que os dados estejam sempre atualizados.

## Estrutura do Projeto

O backend está organizado da seguinte forma, usando as APIs [Querido Diário](https://queridodiario.ok.org.br/api/docs#/) e [Dados Abertos](https://dadosabertos.compras.gov.br/swagger-ui/index.html#/):

- `compras_automatico.py`: Realiza a busca de compras hospitalares na API Querido Diário e salva os dados filtrados em `resultados_compras.json`.
- `credito_automatico.py`: Realiza a busca creditos suplementares na API Querido Diário e salva os dados filtrados em `resultados_credito.json`.
- `verbas_automatico.py`: Realiza a busca verbas parlamentares para escolas na API Querido Diário e salva os dados filtrados em `resultados_verbas.json`.
- `convenio_automatico.py`: Realiza a busca convenios hospitalares na API Querido Diário e salva os dados filtrados em `resultado4.json`.
- `licitacoes_automatico.py`: Realiza a busca licitaçoes na API Dados Abertos e salva os dados filtrados em `resultado5.json`.

Cada algoritmo foi desenvolvido para atender a uma necessidade específica de filtragem e manipulação dos dados retornados pela API.



## Configurando a API
### Querido Diário
#### Usando como exemplo o algoritomo [Verbas_automatico.py](https://github.com/unb-mds/Gastos-DF-Squad12/blob/main/web/Bases%20web/jsons/verbas_automatico.py)

A API do querido diario le os diarios oficiais no formato .txt< desta forma é pesquisado palavras chaves que tem relaçoes com o objeto de pesquisa, como neste exemplo "Nº CRE / UE Capital Custeio Total"

PDF do diario oficial:

![image](https://github.com/user-attachments/assets/a6af53f7-10bc-41b1-b258-94378953d21e)

.txt lida pela API:

![image](https://github.com/user-attachments/assets/fe7ff7f1-b95c-4b43-841b-5e4410094978)




Desta forma, primeiro é nescessario ditar os parâmetros na opção de gazetas:
+ ID do município que deseja realizar a busca
+ Intervalo de tempo em que deseja buscar
+ String que deseja
+ o numero de caractéres que irá retornar
+ Numero maximo de resultados
+ Numero maximo de resultaods por gazeta retornada
+ Ordenar a Data

![image](https://github.com/user-attachments/assets/90652d69-4a82-4ebd-94ad-5845dae0ba1f)



Ao executar a APi ela retornará o seguinte link
~~~
https://queridodiario.ok.org.br/api/gazettes?published_since=2024-01-01&published_until=2024-08-01&querystring=%22N%C2%BA%20CRE%20%2F%20UE%20Capital%20Custeio%20Total%22&excerpt_size=500&number_of_excerpts=1&pre_tags=&post_tags=&size=10&sort_by=relevance
~~~~

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
  
~~~json
{
            "territory_id": "5300108",
            "date": "2021-01-04",
            "scraped_at": "2021-12-16T18:13:08.483389",
            "url": "https://querido-diario.nyc3.cdn.digitaloceanspaces.com/5300108/2021-01-04/92ee5a33891e358bdaee1237577f239a4359d183.pdf",
            "territory_name": "Brasília",
            "state_code": "DF",
            "excerpts": [
                "execução completa do recurso no\nexercício referente ao primeiro pagamento, a sua utilização ficará condicionada a\nautorização da SUPLAV.\nArt. 8º Esta Portaria entra em vigor na data de sua publicação.\n\nFÁBIO PEREIRA DE SOUSA\n \n\nANEXO ÚNICO\nNº CRE / UE Capital Custeio Total\n\n1 CRE GAMA R$ 180.000,00 R$ 0,00 R$ 180.000,00\n\n2 CRE GUARÁ R$ 0,00 R$ 80.000,00 R$ 80.000,00\n\n TOTAL R$ 180.000,00 R$ 80.000,00 R$ 260.000,00\n\nSECRETARIA DE ESTADO\nDE SEGURANÇA PÚBLICA\n\nPORTARIA Nº 135, DE 21 DE DEZEMBRO DE 2020\nO SECRETÁRIO"
            ],
            "is_extra_edition": false,
            "txt_url": "https://querido-diario.nyc3.cdn.digitaloceanspaces.com/5300108/2021-01-04/92ee5a33891e358bdaee1237577f239a4359d183.txt"
        }


~~~

O algoritimo automarico irá sempre buscar o diario oficial da data anterior

~~~python
#pegar a data de ontem
yesterday = datetime.now() - timedelta(days=1)
yesterday_str = yesterday.strftime("%Y-%m-%d")

# Fazendo a solicitação GET para a API
url = f'https://queridodiario.ok.org.br/api/gazettes?territory_ids=5300108&published_since={yesterday_str}&published_until={yesterday_str}&querystring=%22N%C2%BA%20UE%20Custeio%20Total%22%20%22N%C2%BA%20CRE%2FUE%20Capital%20Custeio%20Total%22&excerpt_size=50000&number_of_excerpts=100&pre_tags=&post_tags=&size=10000&sort_by=ascending_date'
response = requests.get(url)
~~~

Após receber o json, o excerpt é tratado e é extraido os valores e os nomes das entidades

~~~python
# Verificando se a solicitação foi bem-sucedida
if response.status_code == 200:
    # Convertendo a resposta para JSON
    dados = response.json()

    # Acessando a lista de gazettes dentro dos dados
    gazettes = dados['gazettes']

    # Iterando sobre cada gazette na lista
    for gazette in gazettes:
        # Acessando a data de cada gazette
        data = gazette['date']
        data_obj = datetime.strptime(data, "%Y-%m-%d")
        data_formatada = data_obj.strftime("%d-%m-%Y")

        # Acessando a lista de excertos de cada gazette
        excertos = gazette['excerpts']

        # Iterando sobre cada excerto
        for excerto in excertos:
            # Função para processar os padrões
            def process_pattern(pattern, regex):
                if pattern in excerto:
                    linhas = excerto.strip().split('\n')
                    for linha in linhas[linhas.index(pattern) + 1:]:
                        matches = re.findall(regex, linha)
                        if matches:
                            for match in matches:
                                escola = match[0].strip()
                                total = match[-1].strip()
                                total_float = float(total.replace('.', '').replace(',', '.'))

                                if escola not in escolas_dict:
                                    escolas_dict[escola] = {"escola": escola, "dados": []}

                                escolas_dict[escola]["dados"].append({
                                    "data": data_formatada,
                                    "valor": total_float
                                })


            # Processando os diferentes padrões
            process_pattern("Nº CRE/UE Capital Custeio Total",
                            r'\d+\s+([\w\s]+?)\s+R\$\s*([\d,.]+)\s+R\$\s*([\d,.]+)\s+R\$\s*([\d,.]+)')
            process_pattern("Nº UE Custeio Total", r'\d+\s+([\w\s]+?)\s+R\$\s*([\d,.]+)\s+R\$\s*([\d,.]+)')
            process_pattern("Nº CRE / UE Capital Custeio Total",
                            r'\d+\s+([\w\s]+?)\s+R\$\s*([\d,.]+)\s+R\$\s*([\d,.]+)\s+R\$\s*([\d,.]+)')
~~~

### Dados Abertos
#### É usado somente em [automatico_licitacoes.py](https://github.com/unb-mds/Gastos-DF-Squad12/blob/main/web/Bases%20web/jsons/licitacoes_automatico.py)

A API retorna um Json com os dados das licitações dentro do periodo de tempo imposto no request
![image](https://github.com/user-attachments/assets/ebc9c880-b2c4-49e4-a8a7-f404a15fe6d8)

retorna jsons como:

~~~json
 {
      "id_compra": "string",
      "identificador": "string",
      "numero_processo": "string",
      "uasg": 0,
      "modalidade": 0,
      "nome_modalidade": "string",
      "numero_aviso": 0,
      "situacao_aviso": "string",
      "tipo_pregao": "string",
      "tipo_recurso": "string",
      "nome_responsavel": "string",
      "funcao_responsavel": "string",
      "numero_itens": 0,
      "valor_estimado_total": 0,
      "valor_homologado_total": 0,
      "informacoes_gerais": "string",
      "objeto": "string",
      "endereco_entrega_edital": "string",
      "codigo_municipio_uasg": 0,
      "data_abertura_proposta": "2024-09-04",
      "data_entrega_edital": "2024-09-04",
      "data_entrega_proposta": "2024-09-04",
      "data_publicacao": "2024-09-04",
      "dt_alteracao": "2024-09-04T04:46:00.753Z"
    }
~~~
Desta forma os dados são colocados no JSON e tratados pelo javascript no frontend

## Atualização Automática dos Arquivos JSON

Um workflow do GitHub foi configurado para rodar automaticamente em um horário determinado ou em intervalos específicos. Este workflow executa os algoritmos e atualiza os arquivos JSON no repositório, garantindo que os dados estejam sempre em sua versão mais recente.

O workflow está definido no arquivo `.github/workflows/update_data.yml` e inclui as seguintes etapas:

1. **Checagem do Repositório**: Verifica se há mudanças pendentes.
2. **Execução dos Algoritmos**: Executa todos os cinco algoritmos.
3. **Atualização dos Arquivos JSON**: Salva os resultados em seus respectivos jsons
4. **Commit e Push**: Faz o commit das mudanças e envia para o repositório.

