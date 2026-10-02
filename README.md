# Pipeline ETL: PostgreSQL, DuckDB e Airflow

Ambiente local com PostgreSQL e Apache Airflow usando Docker Compose. O banco fica disponível no host para ferramentas como pgAdmin.

> **Estado atual:** este repositório prepara a infraestrutura. Ainda não há DAGs ETL em `dags/` nem um serviço DuckDB configurado.

## Pré-requisitos

- Docker Desktop instalado e em execução, com Docker Compose v2.
- Python instalado, usado abaixo para gerar senhas aleatórias.
- pgAdmin, opcional, para explorar o banco graficamente.

## Configuração no Windows (PowerShell)

1. Clone o projeto e entre na pasta:

   ```powershell
   git clone https://github.com/lucasCarvalho2508/pipeline-etl-postgres-duckdb-airflow.git
   Set-Location pipeline-etl-postgres-duckdb-airflow
   ```

2. Crie seu arquivo local de configuração. Ele não deve ser enviado ao Git:

   ```powershell
   Copy-Item .env.example .env
   ```

3. Gere dois segredos hexadecimais independentes:

   ```powershell
   python -c "import secrets; print(secrets.token_hex(24))"
   ```

   Execute o comando duas vezes. Abra `.env` e substitua os valores de
   `POSTGRES_PASSWORD` e `AIRFLOW_ADMIN_PASSWORD` pelos resultados. Em
   `AIRFLOW_DATABASE_URL`, substitua `URL_ENCODED_PASSWORD` pelo valor de
   `POSTGRES_PASSWORD` e mantenha usuário, banco e host consistentes com as
   variáveis `POSTGRES_USER` e `POSTGRES_DB`. Os segredos hexadecimais não
   precisam de codificação adicional para URL. Se escolher uma senha com
   caracteres especiais, codifique-a para URL (por exemplo, `@` vira `%40`).

4. Gere uma chave Fernet e copie o resultado para `AIRFLOW_FERNET_KEY` em `.env`:

   ```powershell
   docker run --rm apache/airflow:2.7.1 python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
   ```

5. Valide a configuração e inicie os serviços:

   ```powershell
   docker compose config --quiet
   docker compose up -d
   ```

6. Confira o estado dos contêineres e a saúde do Airflow:

   ```powershell
   docker compose ps
   curl.exe http://localhost:8081/health
   ```

   A resposta de saúde deve incluir o metadatabase e o scheduler como `healthy`.

## Acessar o Airflow

Abra <http://localhost:8081> e entre com `AIRFLOW_ADMIN_USERNAME` e
`AIRFLOW_ADMIN_PASSWORD`, definidos no `.env`.

Os arquivos de DAG ficam na pasta `dags/`, montada no contêiner em
`/opt/airflow/dags`.

## Acessar o PostgreSQL pelo pgAdmin

No pgAdmin, registre um servidor. Na aba **General**, escolha um nome; na aba
**Connection**, use:

| Campo | Valor |
| --- | --- |
| Host name/address | `localhost` |
| Port | `5434` |
| Maintenance database | valor de `POSTGRES_DB` no `.env` (padrão: `novadrive`) |
| Username | valor de `POSTGRES_USER` no `.env` (padrão: `etlreadonly`) |
| Password | valor de `POSTGRES_PASSWORD` no `.env` |

Salve e abra **Databases → novadrive → Schemas → public**. Clique com o botão
direito em **Schemas** e escolha **Refresh** se precisar atualizar a árvore.
O pgAdmin é um cliente separado; ele não é iniciado pelo Compose.

O mesmo banco também pode ser acessado por outros clientes PostgreSQL em
`localhost:5434`. De dentro dos contêineres, o hostname do banco é `postgres`
e a porta é `5432`.

## Comandos úteis

```powershell
docker compose logs -f
docker compose stop
docker compose start
docker compose down
```

`docker compose down` preserva os dados do PostgreSQL no volume `postgres_data`.
Para apagar também o banco e todos os dados persistidos, use
`docker compose down -v` — essa ação é destrutiva.

Se alterar `POSTGRES_DB`, `POSTGRES_USER` ou `POSTGRES_PASSWORD` depois da
primeira inicialização, o volume existente não é reconfigurado automaticamente.
Altere as credenciais no próprio PostgreSQL ou, se puder descartar os dados,
remova o volume com `docker compose down -v` e inicie os serviços novamente.

## Segredos e Git

As credenciais e a chave Fernet devem ficar somente no `.env`, que é ignorado
pelo Git. `.env.example` contém apenas valores fictícios para orientar a
configuração. Nunca publique senhas ou chaves reais.
