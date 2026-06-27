# 🚀 GLPI Production Stack

Este repositório contém a configuração e o guia de deploy para o sistema GLPI 10 utilizando Docker Compose.

## 📂 Estrutura do Projeto

- `glpi-prod/`: Pasta principal com os arquivos de orquestração.
  - `docker-compose.yml`: Configuração dos serviços.
  - `config/`: Configurações persistentes do PHP/GLPI.
  - `files/`: Anexos, sessões e logs.
  - `marketplace/`: Plugins instalados.
  - `mysql/`: Dados do banco de dados MariaDB.
- `.env`: Variáveis de ambiente e senhas (não versionar dados reais).

---

## 🛠️ Pré-requisitos

Antes de iniciar, certifique-se de que o ambiente possui:
1. **Docker** e **Docker Compose** instalados.
2. Rede Docker `stack_network` criada (necessária para o compose).

---

## 🚀 Como Subir o Sistema (Passo a Passo)

### 1. Preparar a Rede
O stack utiliza uma rede externa chamada `stack_network`. Se ela ainda não existir, crie-a com:
```powershell
docker network create stack_network
```

### 2. Configurar Variáveis
Certifique-se de que o arquivo `.env` na raiz contém as credenciais desejadas.

### 3. Iniciar os Containers
Na raiz do projeto, execute:
```powershell
docker-compose up -d
```

### 4. Acessar o Sistema
Abra seu navegador em:
**URL:** [http://localhost:8080](http://localhost:8080)

### 📝 Assistente de Instalação (Web Wizard)
Durante a instalação no navegador, preencha os dados do banco exatamente assim:
- **Servidor SQL (Hospedeiro)**: `glpi-db` (**IMPORTANTE**: Não use `localhost` ou o IP do servidor).
- **Usuário SQL**: `glpiuser`
- **Senha SQL**: (A definida no seu `.env`)
- **Banco de Dados**: Selecionar `glpidb` (já criado).

---

## 🔐 Credenciais Padrão do GLPI

O GLPI cria os seguintes usuários por padrão. **Altere as senhas no primeiro acesso!**

| Usuário | Senha | Perfil |
| :--- | :--- | :--- |
| **glpi** | **glpi** | Administrador |
| **tech** | **tech** | Técnico |
| **normal** | **normal** | Usuário Comum |
| **post-only** | **post-only** | Apenas abertura de Tickets |

---

## 💾 Manutenção e Backup

### Fazer Backup
Para garantir a segurança dos seus dados, faça cópia das pastas `mysql`, `config`, `files` e `marketplace`.

### Atualizar o Sistema
1. Altere a `GLPI_VERSION` no arquivo `.env`.
2. Rode `docker-compose pull` e depois `docker-compose up -d`.
3. Siga as instruções de atualização na interface web do GLPI.

---

## ⚙️ Configurações Aplicadas (Best Practices)
- **GLPI Version**: Fixada em `11.0.6` (evita quebras automáticas).
- **PHP Memory Limit**: `512M`.
- **Upload Max Size**: `32M`.
- **Database**: MariaDB 10.11 LTS.
- **Log Rotation**: Máximo de 3 arquivos de 10MB para economizar disco.

---

## 🌐 Deploy em VPS (Servidor Remoto)

### 1. Conectar na VPS via SSH
```bash
ssh root@seu-ip-vps
```

### 2. Clonar o Repositório
```bash
cd ~
git clone https://github.com/davi-ricardo/GLPI.git
mv GLPI /opt/glpi
cd /opt/glpi
```

### 3. Criar o Arquivo `.env`
O arquivo `.env` não é versionado por segurança. Crie-o com:
```bash
cat > .env << 'EOF'
# MariaDB Configuration
DB_ROOT_PWD=change_me
GLPI_DB_NAME=glpidb
GLPI_DB_USER=glpiuser
GLPI_DB_PASSWORD=change_me

# GLPI Configuration
GLPI_VERSION=11.0.6
EOF
```

### 4. Preparar Diretórios e Rede
```bash
docker network create stack_network
mkdir -p /opt/glpi/glpi-prod/{mysql,config,files,marketplace}
chmod -R 777 /opt/glpi/glpi-prod/{config,files,marketplace}
```

### 5. Subir os Containers
```bash
docker compose up -d
```

### 6. Acessar o GLPI
Abra no navegador:
```
http://seu-ip-vps:8001
```

### 7. Configurar o Banco de Dados no Assistente
Na tela de instalação, preencha:
- **Servidor SQL**: `glpi-db`
- **Usuário SQL**: `glpiuser`
- **Senha SQL**: `change_me` (ou a do seu `.env`)
- **Banco de Dados**: `glpidb`

### 8. Fazer Login Padrão
- **Usuário**: `glpi`
- **Senha**: `glpi`

⚠️ **Importante**: Altere as credenciais padrão imediatamente após o primeiro acesso.

### Comandos Úteis na VPS

**Verificar status dos containers:**
```bash
cd /opt/glpi
docker compose ps
```

**Ver logs do GLPI:**
```bash
docker compose logs glpi -f --tail=100
```

**Parar o stack:**
```bash
docker compose down
```

**Reiniciar o stack:**
```bash
docker compose restart
```

**Atualizar a versão do GLPI:**
```bash
# Altere a versão no .env
GLPI_VERSION=11.0.7

# Reapply
docker compose pull
docker compose up -d
```

---

## ❓ Solução de Problemas (Troubleshooting)

### Erro: "Access denied for user 'glpiuser'"
Se o banco de dados recusar a senha mesmo ela estando correta no `.env`:
1. O MariaDB grava a senha apenas no **primeiro boot**. Se você alterou o `.env` depois disso, a senha antiga continua valendo no banco.
2. Para forçar a atualização (em instalações novas), pare o container e apague o conteúdo da pasta `glpi-prod/mysql/`, depois suba o container novamente.

### Erro: "Permission Denied" em pastas
Certifique-se de que as pastas `config`, `files` e `marketplace` pertencem ao usuário do webserver (UID 33):
`sudo chown -R 33:33 glpi-prod/config glpi-prod/files glpi-prod/marketplace`
