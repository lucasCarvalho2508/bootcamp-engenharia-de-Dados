# Pipeline ETL: PostgreSQL, DuckDB e Airflow

## Configuração local

1. Copie `.env.example` para `.env`.
2. Defina senhas fortes para o PostgreSQL e para o usuário administrador do Airflow.
3. Configure `AIRFLOW_DATABASE_URL` com as credenciais do PostgreSQL e a senha codificada para URL.
4. Gere uma chave Fernet válida:

   ```powershell
   docker run --rm apache/airflow:2.7.1 python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
   ```

5. Inicie os serviços:

   ```powershell
   docker compose up -d
   ```

O Airflow fica disponível em `http://localhost:8081`; o PostgreSQL é exposto em `localhost:5434`.
Não versione o arquivo `.env`.