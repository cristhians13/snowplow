# Snowplow
Como criar a infraestrutura no GCP utilizando Terraform

# Pré-Requisitos
- Conta na Google Cloud
- Google Cloud SDK instalado: https://cloud.google.com/sdk/docs/install
- Terraform instalado
- Igluctl: https://docs.snowplow.io/docs/pipeline-components-and-applications/iglu/igluctl-2/
    wget https://github.com/snowplow-incubator/igluctl/releases/download/0.10.1/igluctl_0.10.1.zip
    unzip igluctl_0.10.1.zip
    mv igluctl /usr/local/sbin
    chmod +x /usr/local/sbin/igluctl

# Google Cloud
- Criar um projeto
- Criar uma "Service Account"
    Role "Owner"
- Criar chave
    "Manage Keys"
    "Add Key"
    "Create new key"
    "JSON"
- Salvar chave em na máquina e utilizar o comando <export GOOGLE_APPLICATION_CREDENTIALS="KEY PATH"> para criar a variável
- Habilitar:
    Compute Engine API
    Cloud Resource Manager API
    Identity and Access Management (IAM) API
    Cloud Pub/Sub API 

# Git
- Clone do iac Terraform do Snowplow
git clone https://github.com/snowplow/quickstart-examples.git

# Terraform - 01 Iglu Server
- Editar arquivo terraform.tfvars
    project_id: ID do projeto no GCP
    region: Região GCP
    ssh_ip_allowlist: IP para acesso via SSH
        IP local: https://duckduckgo.com/
        Pesquise ip address
    ssh_key_pairs: usuário e chave para acesso via SSH
        Executar o comando <ssh-keygen -t rsa -b 4096> para gerar as chaves públicas e privadas
        user_name: usuário
        public_key: chave pública gerada
    iglu_db_name: nome do db
    iglu_db_username: usuário db
    iglu_db_password: senha db
    iglu_super_api_key: API Key para acessar o server
        https://duckduckgo.com/
        Pesquise uuid
- Executar comandos:
    terraform init
    terraform plan -out plan --var-file=terraform.tfvars
    terraform apply plan
        * Irá gerar um output com o IP do Server

# Igluctl
Para o pipeline funcionar devem ser inseridos os schemos padrão Snowplow no Iglu Server, alterando no comando abaixo com os valores do IP server e API Key
    git clone https://github.com/snowplow/iglu-central
    cd iglu-central
    igluctl static push --public schemas/ http://CHANGE-TO-MY-IGLU-URL.elb.amazonaws.com 00000000-0000-0000-0000-000000000000

# Terraform - 02 Pipeline
- Editar arquivo terraform.tfvars
    Alterar mesmas variáves do 01 Iglu Server
    iglu_server_dns_name: IP gerado pelo output acima
    Configuração de IPs para acesso ao DB
        pipeline_db_authorized_networks = [
            {
                name: nome network
                value: IPs que terão acesso
            }
        ]
- Executar comandos:
    terraform init
    terraform plan -out plan --var-file=terraform.tfvars
    terraform apply plan
        * Irá gerar um output com o IP Collector, IP db e porta

# Destruir recursos
Caso seja necessário destruir a infra na GCP
    terraform destroy --var-file=terraform.tfvars
    1º no 02 Pipelines
    2º no 01 Iglu Server