# Banco API - Testes de Performance com k6

Repositório dedicado aos testes de carga e performance da **Banco API**, desenvolvidos em **JavaScript** utilizando a ferramenta **[Grafana k6](https://k6.io/)**.


## 1. Introdução

Este projeto tem como objetivo avaliar o comportamento, estabilidade, tempo de resposta e resiliência dos principais endpoints da API bancária sob diferentes condições de tráfego.

Através de testes automatizados com o **k6**, simulamos cenários de usuários simultâneos (VUs), definimos estágios de ramp-up, sustentação de carga e ramp-down, além de aplicar *thresholds* (critérios de aceitação) e validações (*checks*) para garantir que os níveis de serviço (SLAs) sejam atendidos antes do envio para ambientes de produção.


## 2. Tecnologias Utilizadas

- **[Grafana k6](https://k6.io/)**: Ferramenta de teste de carga moderna, orientada a desenvolvedores e altamente performática.
- **[JavaScript (ES6+)](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript)**: Linguagem utilizada para a escrita e estruturação dos scripts de teste.
- **JSON**: Formato para massa de dados estática e configurações de ambiente.
- **Git & GitHub**: Versionamento de código e colaboração.


## 3. Estrutura do Repositório

```plaintext
banco-api-performance/
├── config/
│   └── config.local.json         # Configurações locais e fallback de URLs
├── fixtures/
│   └── postLogin.json            # Massa de dados de entrada (payloads)
├── helpers/
│   └── autenticacao.js           # Funções auxiliares (obtenção de token, etc.)
├── tests/
│   ├── login.test.js             # Teste de performance do endpoint de login
│   └── transferencias.test.js    # Teste de performance de transferências bancárias
├── utils/
│   └── variaveis.js              # Utilitário para resolução dinâmica de variáveis de ambiente
├── .gitignore                    # Arquivos e relatórios ignorados no versionamento
└── README.md                     # Documentação do projeto
```


## 4. Objetivo de Cada Grupo de Arquivos

- **`config/`**:
  Contém as configurações base da aplicação por ambiente. No arquivo `config.local.json`, é definida a URL padrão de fallback (`baseUrl: "http://localhost:3000"`), permitindo que os testes sejam executados localmente sem obrigatoriedade de parâmetros adicionais.

- **`fixtures/`**:
  Armazena as massas de dados estáticas em formato JSON (por exemplo, `postLogin.json`). Separa os dados das requisições da lógica dos testes, garantindo manutenibilidade e reutilização dos payloads.

- **`helpers/`**:
  Contém funções auxiliares e regras de negócio reutilizáveis entre os testes. Exemplo: `autenticacao.js` realiza a chamada de autenticação e devolve o token JWT necessário para testar endpoints protegidos.

- **`tests/`**:
  Contém os scripts de teste executáveis pelo k6:
  - `login.test.js`: Simula múltiplos usuários simultâneos autenticando no sistema com controle de estágios de carga (`stages`) e validação de tempo de resposta e taxa de falhas (`thresholds`).
  - `transferencias.test.js`: Testa a rota protegida de transferências bancárias utilizando o token dinâmico gerado pelo helper de autenticação.

- **`utils/`**:
  Reúne funções utilitárias do projeto. O arquivo `variaveis.js` é responsável por ler a variável `BASE_URL` fornecida via linha de comando/ambiente (`__ENV.BASE_URL`) e, caso não exista, aplicar o valor padrão presente em `config.local.json`.


## 5. Modo de Instalação

### Pré-requisitos

É necessário ter o **k6** instalado em sua máquina e o **Git** para clonar o repositório.

### Instalando o k6

Escolha o método de acordo com o seu sistema operacional:

#### Windows
Utilizando o **winget**:
```powershell
winget install k6 --source winget
```

Ou utilizando o **Chocolatey**:
```powershell
choco install k6
```

#### macOS
Utilizando o **Homebrew**:
```bash
brew install k6
```

#### Linux (Debian/Ubuntu)
```bash
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

### Clonando o Repositório

```bash
git clone https://github.com/thaliamariana/banco-api-performance.git
cd banco-api-performance
```


## 6. Modo de Execução do Projeto

### Variável de Ambiente `BASE_URL`

O projeto foi construído para permitir a execução contra qualquer ambiente (local, desenvolvimento, homologação ou produção).

Para informar o endereço da API alvo, utilize o argumento `-e BASE_URL="<URL_DO_AMBIENTE>"` ao invocar o k6. Caso o argumento não seja informado, a URL padrão configurada em `config/config.local.json` (`http://localhost:3000`) será utilizada automaticamente.

---

### Execuções Básicas

#### 1. Executando o teste de login com a URL padrão (localhost)
```bash
k6 run ./tests/login.test.js
```

#### 2. Executando informando a variável `BASE_URL`
```bash
k6 run -e BASE_URL="http://localhost:3000" ./tests/login.test.js
```

#### 3. Executando o teste de transferências
```bash
k6 run -e BASE_URL="http://localhost:3000" ./tests/transferencias.test.js
```

---

### Acompanhamento em Tempo Real e Exportação de Relatórios HTML

O k6 possui o recurso nativo **Web Dashboard**, que permite acompanhar as métricas em tempo real no navegador durante a execução do teste e, ao final, exportar um relatório HTML interativo completo.

#### Opção A: Usando a flag `--out` (Recomendado)

Você pode iniciar o dashboard web e definir o arquivo de exportação diretamente pelo comando:

```bash
k6 run --out "web-dashboard=export=html-report.html" -e BASE_URL="http://localhost:3000" .\tests\login.test.js
```

> **Nota:** Durante a execução, o k6 disponibiliza um painel em tempo real acessível pelo navegador (geralmente em `http://127.0.0.1:5665`). Ao término do teste, o arquivo estático `html-report.html` será salvo na raiz do projeto.

---

#### Opção B: Usando variáveis de ambiente do próprio k6

O k6 permite controlar o dashboard através das suas variáveis de ambiente nativas:
- `K6_WEB_DASHBOARD`: define se o dashboard em tempo real estará ativo (`true` / `false`).
- `K6_WEB_DASHBOARD_EXPORT`: caminho e nome do arquivo HTML onde o relatório será salvo.
- `K6_WEB_DASHBOARD_OPEN`: se definido como `true`, abre a aba no navegador automaticamente.

##### No Windows (PowerShell):
```powershell
$env:K6_WEB_DASHBOARD="true"
$env:K6_WEB_DASHBOARD_EXPORT="html-report.html"
k6 run -e BASE_URL="http://localhost:3000" .\tests\login.test.js
```

##### No Windows (Prompt de Comando / CMD):
```cmd
set K6_WEB_DASHBOARD=true
set K6_WEB_DASHBOARD_EXPORT=html-report.html
k6 run -e BASE_URL="http://localhost:3000" .\tests\login.test.js
```

##### No Linux / macOS (Bash / Zsh):
```bash
K6_WEB_DASHBOARD=true K6_WEB_DASHBOARD_EXPORT="html-report.html" k6 run -e BASE_URL="http://localhost:3000" ./tests/login.test.js
```

---

### Visualizando o Relatório Exportado

Após a conclusão da execução, você pode abrir o relatório `html-report.html` diretamente em seu navegador favorito:

- **Windows (PowerShell)**:
  ```powershell
  Start-Process html-report.html
  ```
- **macOS**:
  ```bash
  open html-report.html
  ```
- **Linux**:
  ```bash
  xdg-open html-report.html
  ```
