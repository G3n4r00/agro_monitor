# Agro Monitor

> Sistema de monitoramento agrícola em tempo real com geração automática de dados, motor de regras booleanas e dashboard web.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Flask](https://img.shields.io/badge/Flask-3.x-lightgrey)
![Pandas](https://img.shields.io/badge/Pandas-2.x-green)
![Status](https://img.shields.io/badge/Status-Ativo-brightgreen)

---

## Visão Geral

Projeto desenvolvido como atividade acadêmica de automação agrícola (FIAP). O sistema simula uma rede de sensores distribuídos em talhões de uma propriedade rural: uma thread em background gera leituras realistas a cada 5 segundos e as persiste em CSV; um motor de regras com lógica booleana avalia continuamente os dados e produz alertas classificados por nível de severidade; e um servidor Flask expõe um dashboard web que se atualiza automaticamente a cada 3 segundos, sem necessidade de recarregar a página.

---

## Funcionalidades

- Geração automática de leituras simuladas de sensores a cada 5 segundos
- Três regras de decisão com lógica booleana (AND / OR) aplicadas em tempo real
- Dashboard web responsivo com atualização automática a cada 3 segundos
- Botões de disparo manual de alertas para demonstração em sala ou apresentação
- Persistência dos dados em CSV com histórico acumulado por execução
- Modal de detalhes por alerta com recomendações de ação e download de relatório HTML
- Interface totalmente self-contained: sem dependências de CDN ou frameworks de frontend

---

## Estrutura do Projeto

```
agro_monitor/
├── main.py          — servidor Flask, thread de geração, dashboard HTML embutido e rotas API
├── regras.py        — motor de regras: três verificações booleanas + método consolidador
├── gerador.py       — geração de leituras normais e forçadas; persistência thread-safe em CSV
└── dados/
    └── leituras.csv — histórico de todas as leituras geradas (criado automaticamente)
```

---

## Regras de Decisão

| Regra | Condição Lógica | Nível | Ação |
|---|---|---|---|
| INFESTAÇÃO | `perc_folhas_doentes > 30` **AND** `praga_detectada != "nenhuma"` | CRÍTICO | Alerta imediato; talhão marcado para monitoramento intensivo |
| IRRIGAÇÃO  | `umidade_solo_pct < 30` **AND** `nivel_irrigacao == "baixo"` | ATENÇÃO | Solicitação de irrigação emergencial; notificação à equipe de manutenção |
| TEMPERATURA | `temperatura_c > 35` **OR** `temperatura_c < 12` | AVISO | Registro de anomalia climática; aumento da frequência de monitoramento |

As condições são avaliadas sobre as últimas 50 linhas do CSV a cada ciclo de 5 segundos.

---

## Instalação e Execução

```bash
# entrar na pasta do projeto
cd agro_monitor

# instalar dependências
pip install flask pandas

# executar
python main.py
```

Abra `http://localhost:5000` no navegador.

---

## Como Usar o Dashboard

### Tabela de leituras em tempo real

A coluna da direita exibe as 20 leituras mais recentes do CSV em ordem decrescente. Cada linha corresponde a uma leitura gerada automaticamente e contém: timestamp, talhão, percentual de folhas doentes, praga detectada (badge colorido), umidade do solo, temperatura e nível de irrigação.

### Cards de alerta

A coluna da esquerda lista os alertas ativos classificados por cor:

| Cor | Nível | Significado |
|---|---|---|
| Vermelho | CRÍTICO | Infestação confirmada — ação imediata necessária |
| Amarelo  | ATENÇÃO | Solo seco com irrigação insuficiente |
| Azul     | AVISO   | Temperatura fora da faixa segura |

Clique em **Ver Detalhes** em qualquer card para abrir o modal com recomendações de ação, ações automáticas realizadas pelo sistema e opção de download do relatório em HTML.

### Botões de disparo manual

O painel superior contém três botões que geram uma leitura com valores garantidamente fora dos limites, forçando o disparo da regra correspondente. Útil para demonstrações sem precisar aguardar que os valores aleatórios normais ultrapassem os limiares.

---

## Detalhes Técnicos

**Linguagem:** Python 3.10+

**Bibliotecas:**

| Biblioteca | Uso |
|---|---|
| `flask` | Servidor web, roteamento e serialização JSON das APIs |
| `pandas` | Leitura e filtragem do CSV, avaliação das máscaras booleanas |
| `threading` | Thread daemon de geração de dados e Lock de acesso ao CSV |
| `csv` / `os` | Criação do arquivo CSV e append thread-safe de novas linhas |

**Thread daemon + Flask:**

Ao iniciar, `main.py` cria uma `threading.Thread(target=loop_geracao, daemon=True)` e a inicia antes de chamar `app.run()`. Por ser daemon, a thread é encerrada automaticamente quando o processo principal (Flask) termina. As duas executam no mesmo processo Python e compartilham as variáveis globais `alertas_recentes` e `csv_lock`.

**threading.Lock:**

O `csv_lock` é criado em `main.py` e injetado em `gerador.py` via `set_lock()`. Ele garante exclusão mútua no acesso ao arquivo CSV: a thread de geração faz append enquanto as rotas Flask leem o arquivo para montar a resposta da API — sem o lock, leituras parciais ou corrupção de dados seriam possíveis sob carga concorrente.

---

## Contexto Acadêmico

Projeto entregue como atividade de automação agrícola, atendendo aos três requisitos da proposta:

1. **Lógica de Decisão** — `regras.py` implementa três regras com operadores booleanos AND e OR aplicados sobre DataFrame pandas
2. **Automação** — `main.py` dispara ações (alertas, registros, notificações simuladas) de forma autônoma a cada ciclo da thread, sem intervenção humana
3. **Integração com Dados** — `gerador.py` produz os dados de entrada e os persiste em `dados/leituras.csv`; o motor de regras os lê e avalia a cada iteração

### Time

<table>
  <tr>
    <th>Nome</th>
    <th>RM</th>
    <th>Turma</th>
  </tr>
  <tr>
    <td>Gabriel Genaro Dalaqua</td>
    <td>551986</td>
    <td>4ESOA</td>
  </tr>
  <tr>
    <td>Alairton Rocha Scabelli </td>
    <td>551454</td>
    <td>4ESOA</td>
  </tr>
  <tr>
    <td>Carolina Nascimento Amorim</td>
    <td>97930</td>
    <td>4ESOA</td>
  </tr>
  <tr>
    <td>Eduardo Marins</td>
    <td>551892</td>
    <td>4ESOA</td>
  </tr>
  <tr>
    <td>Sarah Ribeiro da Silva</td>
    <td>97747</td>
    <td>4ESOA</td>
  </tr>
</table>
