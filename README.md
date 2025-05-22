# API de Originação de Precatórios - Mercatório

Esta API simula a etapa de originação de precatórios na Mercatório, permitindo o cadastro de credores, seus respectivos precatórios, upload de documentos pessoais e a obtenção de certidões de forma manual ou automática.

## Requisitos do Sistema

- Python 3.8+
- Pip
- libmagic (biblioteca do sistema necessária para validação de arquivos)
- Docker e Docker Compose (opcional, para execução em containers)

## Instalação e Execução Local

### 1. Instale as dependências do sistema

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install -y libmagic1

# Windows
# Para Windows, o python-magic-bin será instalado automaticamente pelo pip
```

### 2. Clone o repositório e acesse a pasta do projeto

```bash
git clone https://github.com/ayro10/mercatorio.git
cd mercatorio
```

### 3. Crie e ative um ambiente virtual

```bash
python -m venv venv
source venv/bin/activate  

# No Windows: venv\Scripts\activate
```

### 4. Instale as dependências Python

```bash
pip install -r requirements.txt
```

### 5. Execute a aplicação

```bash
python run.py
```

A aplicação estará disponível em http://localhost:5000

## Execução com Docker

### 1. Certifique-se de ter Docker e Docker Compose instalados

### 2. Construa e inicie os containers

```bash
docker-compose up --build
```

A aplicação estará disponível em http://localhost:5000

### 3. Para parar os containers

```bash
docker-compose down
```

## Estrutura da API

A API possui os seguintes endpoints principais:

### API Principal

- `POST /api/credores` - Cadastra credor e seu precatório
- `GET /api/credores/:id` - Consulta geral do credor, precatório, documentos e certidões
- `POST /api/credores/:id/documentos` - Upload de documentos pessoais
- `POST /api/credores/:id/certidoes` - Upload manual de certidões
- `POST /api/credores/:id/buscar-certidoes` - Simula consulta de certidões via API mock

### API Mock

- `GET /api/certidoes?cpf_cnpj=00000000000` - Simula a consulta de certidões em sistemas externos

## Como Testar a API


### Usando a Interface Web

A aplicação também possui uma interface web para facilitar o uso:
1. Acesse http://localhost:5000 no navegador
2. Use o formulário para cadastrar credores, fazer upload de documentos e certidões

## Detalhes da Implementação

### Entidades Principais

1. **Credor**
   - id, nome, cpf_cnpj, email, telefone

2. **Precatório**
   - id, credor_id, numero_precatorio, valor_nominal, foro, data_publicacao

3. **Documento Pessoal**
   - id, credor_id, tipo (identidade, comprovante_residencia, etc.), arquivo_url, enviado_em

4. **Certidão**
   - id, credor_id, tipo (federal, estadual, municipal, trabalhista), origem (manual, api), arquivo_url ou conteudo_base64, status (negativa, positiva, invalida, pendente), recebida_em

### Banco de Dados

A aplicação utiliza SQLite como banco de dados, armazenado em `instance/mercatorio.db`.

### Uploads de Arquivos

Os arquivos enviados são armazenados na pasta `uploads/` com a seguinte estrutura:
- `uploads/credor_{id}/` - Documentos pessoais
- `uploads/credor_{id}/certidoes/` - Certidões

## Exemplos de Requisições

### 1. Cadastrar Credor e Precatório

```bash
curl -X POST http://localhost:5000/api/credores \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Maria Silva",
    "cpf_cnpj": "12345678900",
    "email": "maria@exemplo.com",
    "telefone": "11999999999",
    "precatorio": {
      "numero_precatorio": "0001234-56.2020.8.26.0050",
      "valor_nominal": 50000.00,
      "foro": "TJSP",
      "data_publicacao": "2023-10-01"
    }
  }'
```

Resposta esperada:
```json
{
  "mensagem": "Credor e precatório cadastrados com sucesso",
  "credor_id": 1,
  "precatorio_id": 1
}
```

### 2. Consultar Credor

```bash
curl -X GET http://localhost:5000/api/credores/1
```

Resposta esperada:
```json
{
  "id": 1,
  "nome": "Maria Silva",
  "cpf_cnpj": "12345678900",
  "email": "maria@exemplo.com",
  "telefone": "11999999999",
  "precatorios": [
    {
      "id": 1,
      "numero_precatorio": "0001234-56.2020.8.26.0050",
      "valor_nominal": 50000.0,
      "foro": "TJSP",
      "data_publicacao": "2023-10-01"
    }
  ],
  "documentos": [],
  "certidoes": []
}
```

### 3. Upload de Documento Pessoal

```bash
curl -X POST http://localhost:5000/api/credores/1/documentos \
  -F "tipo=identidade" \
  -F "arquivo=@/caminho/para/documento.pdf"
```

Resposta esperada:
```json
{
  "mensagem": "Documento enviado com sucesso",
  "documento_id": 1
}
```

### 4. Upload Manual de Certidão

```bash
curl -X POST http://localhost:5000/api/credores/1/certidoes \
  -F "tipo=federal" \
  -F "status=negativa" \
  -F "arquivo=@/caminho/para/certidao.pdf"
```

Resposta esperada:
```json
{
  "mensagem": "Certidão manual enviada com sucesso",
  "certidao_id": 1
}
```

### 5. Buscar Certidões Automaticamente

```bash
curl -X POST http://localhost:5000/api/credores/1/buscar-certidoes
```

Resposta esperada:
```json
{
  "mensagem": "Certidões buscadas e salvas com sucesso",
  "total": 2
}
```

### 6. Consultar API Mock de Certidões

```bash
curl -X GET "http://localhost:5000/api/certidoes?cpf_cnpj=12345678900"
```

Resposta esperada:
```json
{
  "cpf_cnpj": "12345678900",
  "certidoes": [
    {
      "tipo": "federal",
      "status": "negativa",
      "conteudo_base64": "Y2VydGlkYW8gdGVzdGU="
    },
    {
      "tipo": "trabalhista",
      "status": "positiva",
      "conteudo_base64": "Y2VydGlkYW8gdGVzdGU="
    }
  ]
}
```

## Executando Testes

Para executar os testes automatizados:

```bash
# Instalar dependências de teste (se ainda não instalou)
pip install pytest pytest-flask

# Executar todos os testes
python -m pytest

# Executar testes específicos
python -m pytest tests/test_uploads.py
python -m pytest tests/test_integracao.py

# Executar com detalhes
python -m pytest -v
```

## Notas Adicionais

- A API mock está configurada para retornar sempre os mesmos dados de certidões para qualquer CPF/CNPJ
- Os uploads de arquivos aceitam apenas PDF, JPG e PNG
- O tamanho máximo de arquivo é de 10MB
- As certidões podem ter status: negativa, positiva, invalida ou pendente
- Os documentos pessoais podem ser do tipo: identidade, comprovante_residencia, ou outros tipos configurados no enum TipoDocumento
- Exitem alguns bugs conhecidos na interface gráfica, como quando a pessoa escolhe um arquivo, e depois troca e da upload no segundo arquivo, os dois são enviados para o banco.


