# Documentação do Workflow: matheus_lucas_teste

## Resumo
Este workflow no **n8n** realiza a captura de e-mails não lidos na caixa "testeonflyrpa@gmail.com", verifica se possuem anexos CSV, envia os arquivos para uma pasta no Google Drive e responde ao remetente informando se os arquivos foram recebidos ou se estão faltando. Além disso, o workflow consulta a cotação do dólar via API para incluir nas respostas. Caso seja enviado mais de um anexo, ele irá armazenar todos, desde que a extensão seja .csv.

O link público da pasta no Google drive onde os arquivos csv são armazenados é: https://drive.google.com/drive/u/2/folders/1HabY3BjRhN-dVhIqFeEuNv6U4UO3ZDid



## Integrações:
- **Google Drive**
  - Foi criado um app no Google Cloud e vinculado API Key e API Secret para autenticação no n8n;
  - A url do n8n foi liberada dentro do App Google cloud para possibilitar a integração.

- **Gmail**
  - Autenticação realizada diretamente no n8n na aba "Credentials".

- **API Para cotação do Dólar**
  - É realizada uma chamada HTTP na rota "https://open.er-api.com/v6/latest/USD"

---

## Nodes

### 1. **Gmail Trigger**
- **Tipo:** `n8n-nodes-base.gmailTrigger`
- **Função:** Monitora a conta do Gmail em busca de novos e-mails não lidos.
- **Parâmetros principais:**
  - `pollTimes`: a cada minuto (`everyMinute`)
  - `filters.readStatus`: `unread`
  - `options.downloadAttachments`: `true`
- **Credenciais:** Conta Gmail OAuth2
- **ID:** `b30a2fc5-56fd-4aed-bbca-c6fb8e087cfc`

---

### 2. **HTTP Request**
- **Tipo:** `n8n-nodes-base.httpRequest`
- **Função:** Consulta a cotação do dólar em relação ao real.
- **URL:** `https://open.er-api.com/v6/latest/USD`
- **ID:** `9be1ee35-7c35-4868-a84e-d10638e8b2f5`

---

### 3. **Code in JavaScript**
- **Tipo:** `n8n-nodes-base.code`
- **Função:** 
  - Verifica os anexos do e-mail.
  - Identifica arquivos CSV.
  - Retorna os nomes dos arquivos, threadId e remetente.
- **Lógica principal:**
  - Se houver CSVs, marca `hasCsv: true` e adiciona arquivos em `results`.
  - Se não houver CSV, retorna `hasCsv: false`.
- **ID:** `658232c2-8163-4c71-9c62-4bf793b04e64`

---

### 4. **If**
- **Tipo:** `n8n-nodes-base.if`
- **Função:** Condicional que verifica se foram encontrados arquivos CSV.
- **Condição:** `{{$json.hasCsv}} == true`
- **ID:** `7c987254-5dc1-4e8d-8fba-aa39a3f19e59`

---

### 5. **Upload file**
- **Tipo:** `n8n-nodes-base.googleDrive`
- **Função:** Faz upload dos arquivos CSV encontrados para uma pasta específica do Google Drive.
- **Parâmetros principais:**
  - `driveId`: "My Drive"
  - `folderId`: `"1HabY3BjRhN-dVhIqFeEuNv6U4UO3ZDid"`
- **Credenciais:** Conta Google Drive OAuth2
- **ID:** `20b2214d-ef95-4441-a61e-f42818b1e611`

---

### 6. **Code in JavaScript1**
- **Tipo:** `n8n-nodes-base.code`
- **Função:** Gera uma lista de arquivos CSV enviados ao Drive, utilizada na resposta ao remetente.
- **ID:** `6190a7fb-f03c-4d67-9758-8c86ef9ebdd0`

---

### 7. **Recebi os arquivos**
- **Tipo:** `n8n-nodes-base.gmail`
- **Função:** Envia um e-mail de confirmação para o remetente informando que os arquivos CSV foram recebidos.
- **Mensagem HTML:** Contém:
  - Saudação personalizada (`{{ $('Gmail Trigger').first().json.from.value[0].name }}`)
  - Cotação do dólar (`{{ $('HTTP Request').first().json.rates.BRL }}`)
  - Lista de arquivos recebidos
  - Rodapé com logotipo
- **ID:** `1c7dea08-825e-4469-a003-6f9032cb4e0e`

---

### 8. **Não recebi os arquivos**
- **Tipo:** `n8n-nodes-base.gmail`
- **Função:** Envia um e-mail ao remetente informando que os anexos CSV não foram recebidos.
- **Mensagem HTML:** Contém:
  - Saudação personalizada
  - Aviso sobre ausência de anexos CSV
  - Cotação do dólar
  - Rodapé com logotipo
- **ID:** `cf05c918-3c33-44ca-bbc5-8ee4d1d63ab5`

---

## Fluxo do Workflow

1. **Gmail Trigger** monitora novos e-mails.
2. **HTTP Request** consulta a cotação do dólar.
3. **Code in JavaScript** verifica se os anexos são CSV.
4. **If** decide o caminho:
   - **True (CSV encontrado):**
     - **Upload file** envia os CSVs para o Google Drive.
     - **Code in JavaScript1** prepara a lista de arquivos.
     - **Recebi os arquivos** envia e-mail de confirmação.
   - **False (CSV não encontrado):**
     - **Não recebi os arquivos** envia e-mail informando ausência de anexos.

---

## Sugestão para testes:

1. Envie um e-mail para: "testeonflyrpa@gmail.com";
2. Caso você envie arquivos csv em anexo, a automação deverá lhe responder o e-mail confirmando o recebimento, a cotação do dólar, citar o nome de cada arquivo csv enviado e lhe direcionar o caminho para a pasta para conferência;
3. Caso não seja enviado nenhum arquivo csv, a automação responderá indicando a falta dos arquivos.

---

## Observações
- Workflow totalmente automatizado.
- Mensagens de e-mail com HTML customizado para melhor visualização e assinatura.
- Integração com Google Drive e Gmail via OAuth2.
- Consulta de cotação do dólar a cada iteração.

---

## Legenda de Nodes e seus respectivos IDs
| Node                    | ID                                      |
|-------------------------|----------------------------------------|
| Gmail Trigger           | b30a2fc5-56fd-4aed-bbca-c6fb8e087cfc  |
| HTTP Request            | 9be1ee35-7c35-4868-a84e-d10638e8b2f5  |
| Code in JavaScript      | 658232c2-8163-4c71-9c62-4bf793b04e64  |
| If                      | 7c987254-5dc1-4e8d-8fba-aa39a3f19e59  |
| Upload file             | 20b2214d-ef95-4441-a61e-f42818b1e611  |
| Code in JavaScript1     | 6190a7fb-f03c-4d67-9758-8c86ef9ebdd0  |
| Recebi os arquivos      | 1c7dea08-825e-4469-a003-6f9032cb4e0e  |
| Não recebi os arquivos  | cf05c918-3c33-44ca-bbc5-8ee4d1d63ab5  |

---

## Autor
**Matheus Lucas**  
Workflow criado para teste e automação de e-mails com anexos CSV.

