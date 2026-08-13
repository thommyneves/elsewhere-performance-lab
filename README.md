# Elsewhere Performance Lab

Projeto para coletar estatísticas competitivas de Valorant no Tracker.gg, gerar análises com proteção contra amostras pequenas e apresentar os resultados em um dashboard interativo.

O projeto possui três etapas:

1. `puxar_tracker.py` coleta dados de agentes e mapas.
2. `analisar_tracker.py` calcula rankings e insights estatísticos.
3. O dashboard exibe o resultado e acompanha automaticamente mudanças no JSON.

## Funcionalidades

- Coleta de estatísticas por agente.
- Coleta de estatísticas por mapa.
- Exportação para CSV, Excel e JSON.
- Top 5 agentes com mínimo de partidas.
- Penalização de desempenhos extremos em amostras pequenas.
- Ranking dos mapas com score conservador.
- Estimativa do melhor agente para cada mapa.
- Dashboard dark com detalhes em roxo neon.
- Atualização automática do dashboard a cada três segundos.
- Interface responsiva para computador e celular.

## Estrutura de pastas

Organize o projeto desta forma:

```text
D:\códigos\trackerprojeto\
├── puxar_tracker.py
├── analisar_tracker.py
├── README.md
│
├── dados_tracker\
│   ├── tracker_dados.json
│   ├── tracker_insights.json
│   ├── tracker_dados.xlsx
│   ├── tracker_agentes.csv
│   ├── tracker_mapas.csv
│   ├── debug_agente.html
│   ├── debug_agente.png
│   ├── debug_mapa.html
│   └── debug_mapa.png
│
└── elsewhere-performance-lab-source\
    ├── app\
    ├── build\
    ├── public\
    │   └── tracker_insights.json
    ├── worker\
    ├── package.json
    ├── package-lock.json
    ├── vite.config.ts
    └── tsconfig.json
```

## Requisitos

### Python

- Python 3.11 ou superior.
- Pip.
- Ambiente virtual recomendado.

### Dashboard

- Node.js 22 ou superior.
- npm.

Confira as instalações:

```powershell
python --version
pip --version
node --version
npm --version
```

## Instalação do coletor

Abra o PowerShell na raiz do projeto:

```powershell
cd "D:\códigos\trackerprojeto"
```

Crie um ambiente virtual:

```powershell
python -m venv .venv
```

Ative o ambiente:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned
& ".\.venv\Scripts\Activate.ps1"
```

Instale as dependências:

```powershell
python -m pip install --upgrade pip
python -m pip install --upgrade playwright beautifulsoup4 pandas openpyxl
python -m playwright install chromium
```

O `analisar_tracker.py` utiliza somente bibliotecas nativas do Python e não exige instalações adicionais.

## Configuração do jogador

No início do `puxar_tracker.py`, confira estas configurações:

```python
RIOT_ID = "elsewhere#999t"
PLATAFORMA = "pc"
PLAYLIST = "competitive"
SEASON_ID = "4f0864e2-40af-28a4-de2c-0e9e64e75f23"
```

Para analisar outro jogador ou outra temporada, altere `RIOT_ID` e `SEASON_ID`.

## Executando a coleta

Na raiz do projeto:

```powershell
python puxar_tracker.py
```

Para utilizar o Google Chrome instalado em vez do Chromium do Playwright:

```powershell
python puxar_tracker.py --usar-chrome
```

Se o Tracker.gg solicitar uma verificação humana, conclua-a na janela aberta e pressione `Enter` no terminal.

O coletor cria os arquivos em:

```text
D:\códigos\trackerprojeto\dados_tracker
```

## Executando a análise

Após a coleta, execute:

```powershell
python analisar_tracker.py
```

O analisador lê:

```text
dados_tracker\tracker_dados.json
```

E salva o mesmo resultado simultaneamente em:

```text
dados_tracker\tracker_insights.json
```

```text
elsewhere-performance-lab-source\public\tracker_insights.json
```

Os caminhos são calculados a partir da localização do próprio `analisar_tracker.py`. Portanto, o script funciona mesmo se o terminal tiver sido aberto em outra pasta.

## Metodologia da análise

O ranking não utiliza somente o win rate bruto. Isso evita que um agente jogado uma única vez com 100% de vitória seja classificado automaticamente como o melhor.

### Agentes

A pontuação considera:

| Estatística | Peso |
|---|---:|
| Win rate | 35% |
| K/D | 25% |
| ADR | 15% |
| ACS | 15% |
| KAST | 10% |

O cálculo aplica:

- Regressão à média.
- Normalização por z-score.
- Fator de confiabilidade baseado no número de partidas.
- Mínimo de cinco partidas para entrar no Top 5.

Agentes com menos partidas continuam no ranking completo, mas são marcados como amostra insuficiente.

### Mapas

O ranking dos mapas considera:

| Estatística | Peso |
|---|---:|
| Win rate conservador | 55% |
| K/D | 20% |
| ADR | 10% |
| ACS | 10% |
| DDΔ | 5% |

O win rate conservador utiliza o limite inferior de Wilson, reduzindo a vantagem artificial de mapas com poucas partidas.

### Melhor agente por mapa

O Tracker.gg informa os agentes de destaque de cada mapa e seus percentuais, mas não informa no arquivo quantas partidas cada agente jogou especificamente naquele mapa.

Por isso, a chance apresentada no dashboard é uma estimativa conservadora, não uma garantia de vitória.

## Instalando o dashboard

Entre na pasta do dashboard:

```powershell
cd "D:\códigos\trackerprojeto\elsewhere-performance-lab-source"
```

Instale todas as dependências, incluindo as de desenvolvimento:

```powershell
npm install --include=dev
```

Confira se o Vite foi instalado:

```powershell
npm list vite
```

O resultado deve incluir uma versão do Vite, por exemplo:

```text
vite@8.0.13
```

## Iniciando o dashboard

Execute:

```powershell
npm run dev
```

O terminal mostrará um endereço semelhante a:

```text
http://localhost:5173/
```

Abra esse endereço no navegador e mantenha o terminal ligado.

## Atualização automática

O dashboard consulta este arquivo a cada três segundos:

```text
elsewhere-performance-lab-source\public\tracker_insights.json
```

Como o `analisar_tracker.py` atualiza esse arquivo automaticamente, basta executar novamente a coleta e a análise.

Use dois terminais.

### Terminal 1 — dashboard

```powershell
cd "D:\códigos\trackerprojeto\elsewhere-performance-lab-source"
npm run dev
```

### Terminal 2 — atualização dos dados

```powershell
cd "D:\códigos\trackerprojeto"
& ".\.venv\Scripts\Activate.ps1"
python puxar_tracker.py
python analisar_tracker.py
```

Após a análise, o dashboard mostrará os novos dados em até três segundos.

## Fluxo rápido de uso

Depois que tudo estiver instalado:

```powershell
cd "D:\códigos\trackerprojeto"
& ".\.venv\Scripts\Activate.ps1"
python puxar_tracker.py
python analisar_tracker.py
```

Em outro terminal:

```powershell
cd "D:\códigos\trackerprojeto\elsewhere-performance-lab-source"
npm run dev
```

## Solução de problemas

### `npm` não é reconhecido

Instale a versão LTS do Node.js:

```powershell
winget install OpenJS.NodeJS.LTS
```

Feche e abra novamente o VS Code ou PowerShell. Depois confira:

```powershell
node --version
npm --version
```

### `WRANGLER_LOG_PATH` não é reconhecido

O `package.json` precisa possuir:

```json
"dev": "vite"
```

Para corrigir automaticamente:

```powershell
npm pkg set scripts.dev="vite"
```

### `vite` não é reconhecido

Instale também as dependências de desenvolvimento:

```powershell
npm install --include=dev
```

Confira o executável:

```powershell
Test-Path ".\node_modules\.bin\vite.cmd"
```

O resultado esperado é `True`.

Se necessário:

```powershell
npm install --save-dev vite@8.0.13
```

### `Could not resolve './build/sites-vite-plugin'`

Confira se este arquivo existe:

```text
elsewhere-performance-lab-source\build\sites-vite-plugin.ts
```

Confira também:

```text
elsewhere-performance-lab-source\worker\index.ts
```

Se estiverem ausentes, extraia novamente o pacote completo para Windows.

### Dashboard mostra “Demonstração”

Isso significa que o dashboard não conseguiu carregar o JSON real. Confira se existe:

```text
elsewhere-performance-lab-source\public\tracker_insights.json
```

Depois execute:

```powershell
cd "D:\códigos\trackerprojeto"
python analisar_tracker.py
```

No topo do dashboard deverá aparecer `JSON conectado`.

### Mapas ou agentes não foram extraídos

Consulte os arquivos de diagnóstico:

```text
dados_tracker\debug_agente.html
dados_tracker\debug_agente.png
dados_tracker\debug_mapa.html
dados_tracker\debug_mapa.png
```

Esses arquivos registram a página carregada pelo Playwright e ajudam a identificar alterações feitas pelo Tracker.gg.

## Arquivos principais

| Arquivo | Responsabilidade |
|---|---|
| `puxar_tracker.py` | Coleta agentes e mapas do Tracker.gg. |
| `analisar_tracker.py` | Calcula rankings, scores e insights. |
| `tracker_dados.json` | Dados brutos estruturados da coleta. |
| `tracker_insights.json` | Resultado final utilizado pelo dashboard. |
| `app/page.tsx` | Componentes e comportamento do dashboard. |
| `app/globals.css` | Identidade visual dark e roxa. |
| `public/tracker_insights.json` | Cópia consultada automaticamente pelo navegador. |

## Observações

- Os endpoints internos do Tracker.gg não são utilizados diretamente.
- O coletor lê os dados renderizados na página.
- Mudanças no HTML do Tracker.gg podem exigir ajustes nos seletores.
- Mantenha os arquivos de diagnóstico enquanto estiver testando novas temporadas ou perfis.
- Não feche o terminal que está executando `npm run dev` enquanto estiver usando o dashboard.

## Autor

Projeto desenvolvido para análise competitiva do jogador `elsewhere#999t`.
