# ⛁ Provisionamento Automatizado do Oracle Database 26ai single instance no filesystem com Ansible

[![GitHub](https://img.shields.io/badge/Repository-danilo01arrudal/ol8-ora26sifs-ansible-blue?logo=github)](https://github.com/danilo01arrudal/ol8-ora26sifs-ansible)
[![Ansible](https://img.shields.io/badge/Ansible-2.9+-black?logo=ansible)](https://www.ansible.com/)
[![Oracle](https://img.shields.io/badge/Oracle-26ai-red?logo=oracle)](https://www.oracle.com/database/)
[![Oracle Linux](https://img.shields.io/badge/Oracle%20Linux-8.10-red?logo=oracle)](https://www.oracle.com/linux/)

---

## 📋 Visão Geral

Este projeto tem como objetivo **automatizar a preparação do ambiente, configuração e instalação base do Oracle Database 26ai Single Instance em ambientes Oracle Linux 8.10**, utilizando **Ansible** como ferramenta de automação.
Baseado na documentação oficial e em manuais técnicos, a solução transforma comandos imperativos de SO (desativação de firewall, configuração de repositórios, setup do kernel) em código declarativo (*Infrastructure as Code*).

- **Reprodutível**: todo o processo é descrito como código (Infrastructure as Code).
- **Flexível**: suporte a diferentes perfis de instância (pequena, média, grande) com parâmetros de performance pré-definidos.
- **Integrado**: pipeline CI/CD via GitHub Actions para execução automatizada.
- **Seguro**: acesso via SSH com chave pública e gerenciamento de secrets.

---

## 🚀 Principais Funcionalidades

| Funcionalidade | Descrição |
|----------------|-----------|
| **Preparação do SO** | Desativação do Firewalld, SELinux e configuração do `clocksource`. |
| **Pré-Requisitos Automáticos** | Instalação do pacote `oracle-ai-database-preinstall-26ai`. |
| **Configuração de Diretórios** | Criação hierárquica automática das pastas `/u01/app/oracle` com as devidas permissões (`oinstall`). |
| **Configuração Dinâmica** | Criação automática de scripts de ambiente (`setEnv.sh`, `start_all.sh`, `stop_all.sh`). |
| **Automação de Serviço** | Configuração do Daemon `dbora.service` no Systemd para inicialização junto com a máquina. |
| **CI/CD com GitHub Actions** | Pipelines para deploy acionados manualmente ou via push. |

---

## 🏗️ Estrutura do Projeto

```plaintext
ol8-oracle26aisifs-ansible/
├── group_vars/
│   └── all/
│       └── vars.yml                # Variáveis comuns (ORACLE_HOME, SID, Base)
├── inventory/
│   └── production                  # Inventário dos hosts gerenciados
├── playbooks/
│   ├── site.yml                    # Playbook principal de provisionamento
│   └── uninstall.yml               # Playbook de rollback/desinstalação
├── roles/
│   └── oracle26ai/
│       ├── tasks/
│       │   ├── main.yml            # Orquestração das tarefas
│       │   ├── pre_reqs.yml        # Configuração SO, SELinux, RPMs
│       │   └── configure.yml       # Criação de scripts, oratab, bash_profile
│       └── templates/
│           ├── setEnv.sh.j2        # Template de export de variáveis
│           ├── start_all.sh.j2     # Script dbstart
│           ├── stop_all.sh.j2      # Script dbshut
│           └── dbora.service.j2    # SystemD service unit
├── ansible.cfg                     # Configuração do Ansible
└── README.md                       # Este arquivo
```

---

## ⚙️ Tecnologias Utilizadas

| Tecnologia | Versão | Finalidade |
|------------|--------|------------|
| **Oracle Linux** | 8.10 | Sistema operacional base (host e alvo). |
| **Oracle Database** | 26 | Sistema de gerenciamento de banco de dados. |
| **Ansible** | 2.9+ | Automação de provisionamento e configuração. |
| **GitHub Actions** | — | Pipeline CI/CD para execução automatizada. |
| **Jinja2** | — | Template engine para arquivos de configuração dinâmicos. |

---

## 🔄 Integração Contínua (CI/CD) com GitHub Actions

O projeto conta com dois workflows configurados no diretório `.github/workflows/`:

### `deploy-postgres.yml`
- **Trigger**: `workflow_dispatch` (execução manual).
- **Executor**: `self-hosted` (runner instalado na própria VM Oracle Linux 8.10).
- **Etapas**:
  1. Checkout do código.
  2. Instalação do Ansible (se necessário).
  3. Execução do playbook `site.yml`.

### `uninstall.yml`
- **Trigger**: `workflow_dispatch` (execução manual).
- **Executor**: `self-hosted`.
- **Etapas**:
  1. Checkout do código.
  2. Execução do playbook `uninstall.yml`.

> **Observação**: Os workflows utilizam um **runner auto-hospedado** configurado na VM alvo, garantindo que a execução ocorra no ambiente Oracle Linux 8.10 real, sem dependência de containers ou emuladores.

---

## 📦 Pré‑requisitos

Antes de executar o projeto, certifique-se de que:

- ✅ A **máquina de controle** (onde o Ansible será executado) possui o Ansible instalado.
- ✅ A **VM alvo** está rodando **Oracle Linux 8.10** com Python 3 instalado.
- ✅ O **acesso SSH** da máquina de controle para a VM alvo está configurado (de preferência com chave pública).
- ✅ O repositório GitHub está clonado localmente ou acessível via CI/CD.

---

## 🛠️ Como Utilizar

### 1. Clonar o repositório
```bash
git clone https://github.com/danilo01arrudal/ol8-ora26sifs-ansible.git
cd ol8-ora26sifs-ansible
```

### 2. Ajustar o inventário
Edite o arquivo `inventory/production` com o IP e usuário da VM alvo:
```ini
[servidores]
ol8-ora26sifs ansible_host=192.168.1.23 ansible_user=root

[servidores:vars]
ansible_python_interpreter=/usr/bin/python3
```

### 3. Escolher a classe da instância
No inventário, adicione a variável `pg_classe` com o valor desejado (`pequena`, `media` ou `grande`):
```ini
ol8-ora26sifs ansible_host=192.168.1.23 ansible_user=root pg_classe=media
```

### 4. Executar o playbook
```bash
ansible-playbook -i inventory/production playbooks/site.yml
```

### 5. Desinstalar (se necessário)
```bash
ansible-playbook -i inventory/production playbooks/uninstall.yml
```

---

## 🔐 Configuração do Runner Auto-Hospedado (para CI/CD)

Para que os workflows do GitHub Actions funcionem, é necessário instalar um runner na VM Oracle Linux 8.10:

```bash
# Criar diretório e baixar o runner
mkdir ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-linux-x64-2.335.1.tar.gz -L https://github.com/actions/runner/releases/download/v2.335.1/actions-runner-linux-x64-2.335.1.tar.gz
tar xzf ./actions-runner-linux-x64-2.335.1.tar.gz

# Configurar (substitua o token pelo fornecido pelo GitHub)
./config.sh --url https://github.com/danilo01arrudal/ol8-ora26sifs-ansible --token SEU_TOKEN

# Instalar como serviço systemd
./svc.sh install
./svc.sh start
```

---

## 📝 Personalização

### Adicionar uma nova classe de instância
1. Crie um diretório em `group_vars/pg_classe_nova/`.
2. Dentro, crie um arquivo `vars.yml` com os parâmetros desejados.
3. No inventário, defina `pg_classe=nova`.

### Alterar parâmetros padrão
Edite o arquivo `roles/postgresql/vars/main.yml` para modificar os valores padrão da role.

---

## 🤝 Contribuição

Contribuições são bem-vindas! Sinta-se à vontade para abrir issues ou pull requests com melhorias, correções ou novas funcionalidades.

---

## 📄 Licença

Este projeto está sob a licença MIT. Consulte o arquivo [LICENSE](LICENSE) para mais detalhes.

---

## 👤 Autor

**Danilo Arruda**  
- GitHub: [@danilo01arrudal](https://github.com/danilo01arrudal)

---

## 🙏 Agradecimentos

- [PostgreSQL Global Development Group](https://www.postgresql.org/) pela excelente base de dados.
- [Ansible](https://www.ansible.com/) pela poderosa ferramenta de automação.
- [Oracle Linux](https://www.oracle.com/linux/) pela plataforma estável e confiável.
```
