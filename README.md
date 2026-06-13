<div align="center">

<img src="assets/banner.png" alt="tj-mapa — mapa estrutural de processos judiciais" width="100%">

<br><br>

<img src="assets/logo.png" alt="tj-mapa" width="92">

<h1>tj-mapa</h1>

<strong>Mapa estrutural de um processo judicial inteiro — em segundos, por centavos.</strong>

<p>
<img src="https://img.shields.io/badge/feito%20em-Go-00ADD8?logo=go&logoColor=white" alt="Go">
<img src="https://img.shields.io/badge/Windows%20·%20Linux%2FWSL-x86--64-1E1E24" alt="Plataformas">
<img src="https://img.shields.io/badge/TecJusti%C3%A7a-OCR%20%E2%86%92%20mapa%20%E2%86%92%20MCP-C9A227" alt="Ecossistema TecJustiça">
</p>

</div>

---

## Em 1 minuto

O `tj-mapa` é **um comando só**. Você dá a ele o processo em texto (o Markdown do OCR) e ele
devolve um **índice** que diz onde está cada peça — petição, decisão, sentença… — com
**página e linha exatas**:

```bash
tj-mapa -in processo.md -formato ambos
```

Para funcionar, ele usa **duas chaves de API** (configura uma vez, leva 30 segundos):

| Chave | O que é | Para quê serve |
|---|---|---|
| 🔑 **OpenRouter** | o **motor** | é a IA que lê os trechos e monta o índice — é quem **faz o trabalho** (você paga o uso: centavos por processo) |
| 🪪 **TecJustiça** | a **licença** | só **confirma que você tem acesso**. É a **mesma chave do OCR** — afinal o tj-mapa depende do OCR pra gerar a entrada |

Em resumo: sem a do **OpenRouter** ele não tem como trabalhar; sem a da **TecJustiça** ele não
libera. Como pegar e configurar as duas está no passo 3, logo abaixo.

---

Não é um resumo. Não é "mais um wrapper de ChatGPT". É um **algoritmo compilado em Go** que
transforma centenas de páginas de autos num índice navegável e preciso, a um custo tão baixo que
você roda em todo processo, todo dia.

Parte do ecossistema **TecJustiça**: OCR jurídico ([ocr.tecjustica.com](https://ocr.tecjustica.com/))
→ **tj-mapa** → MCP ([mcp.tecjustica.com](https://mcp.tecjustica.com/)) — para usar com o Claude Code.

---

## "Mas isso não é só um índice que qualquer um faz?"

Não. Montar um índice jogando o processo num **workflow de agentes** (LangGraph e afins) é fácil
— e é **lento, caro e instável**: relê o documento inteiro, faz dezenas de chamadas ao modelo,
gasta dólares e ainda **alucina** os números de página e linha. A cada nova pergunta, recomeça.

O `tj-mapa` foi para o outro lado. Mesma tarefa, ordem de grandeza diferente:

| | Workflow de agentes (a forma "óbvia") | **tj-mapa** |
|---|---|---|
| Motor | LLM relendo tudo, muitas idas e voltas | **algoritmo em Go**, determinístico e local + IA cirúrgica |
| Processo de ~300 págs | ~centenas de chamadas · ~900 mil tokens | **~12 chamadas · ~48 mil tokens** |
| Custo por processo | dólares (e sobe com o uso) | **centavos** |
| Tempo | minutos a horas | **estrutura em segundos** |
| Página/linha de cada peça | o modelo **chuta e alucina** | **exatas** — calculadas por código |
| Repetir a execução | resultado muda | **reproduzível** |

A diferença não é "um prompt melhor". É **arquitetura**.

## O diferencial: a inteligência está no algoritmo (em Go), não no LLM

O núcleo do `tj-mapa` é um **binário compilado em Go** — sem runtime, sem dependências, sem
orquestração de agentes. Ele faz o trabalho pesado de forma **determinística e local, em
milissegundos**, e só consulta um modelo de IA **no ponto exato que exige julgamento**, enviando
**pedaços minúsculos** de texto (é por isso que custa centavos, e não dólares).

> O segredo do mapeamento está na **engenharia do nosso algoritmo** — proprietário — não em
> "jogar tudo num LLM e torcer". O modelo é uma peça barata e substituível; a inteligência que
> coordena o processo é o que entrega **velocidade + custo mínimo + precisão ao mesmo tempo**,
> e em escala. Qualquer um faz um índice; fazer **assim** é o nosso negócio.

## O que ele entrega

Para **cada peça** do processo (petição, contestação, despacho, decisão, sentença, certidão,
ofício, mandado…), o mapa diz:

- **Onde está** — intervalo exato de **página** e de **linha**;
- **O que é** — o tipo provável da peça;
- **Do que trata** — um resumo de uma ou duas frases;
- **A prova** — uma citação literal que ancora o trecho no original.

O original continua sendo a fonte da verdade — o mapa só o torna **navegável**.

## O mapa na prática

```text
$ tj-mapa -in processo.md -formato ambos

  ✓ 312 páginas  →  47 peças mapeadas  ·  1m54s  ·  custo: US$ 0,04
  → mapa.json  /  mapa.md     (a estrutura sozinha sai em segundos: -sem-enriquecer)
```

Um trecho do `mapa.md`:

| # | Páginas | Linhas | Peça | Resumo |
|---|--------|--------|------|--------|
| 2 | 2–5 | 46–277 | PETIÇÃO INICIAL | Denúncia do MP por furto qualificado. |
| 4 | 8–9 | 412–489 | DECISÃO | Recebimento da denúncia; designa audiência. |
| 23 | 142–148 | 13.820–14.310 | SENTENÇA | Condena o réu; pena em regime aberto. |

**O ganho para o agente de IA:**

> **Antes:** lê ~28.000 linhas para localizar a sentença — caro e lento.
> **Depois:** consulta o mapa, vê *"SENTENÇA — págs 142–148, linhas 13.820–14.310"* e lê **490 linhas**.

## De onde vem a entrada: o OCR jurídico da TecJustiça

<div align="center">
<img src="assets/fluxo.png" alt="Fluxo: PDF dos autos → OCR jurídico → tj-mapa → mapa navegável" width="92%">
</div>

O `tj-mapa` **não lê PDF** — ele parte do **Markdown gerado pelo OCR jurídico da TecJustiça**.
É uma **dependência** do fluxo:

```
   PDF dos autos
        │   OCR jurídico  →  ocr.tecjustica.com
        ▼
   Markdown  (uma página por marcador "## Página N")
        │   tj-mapa
        ▼
   mapa.json  +  mapa.md   →  o agente de IA navega direto ao ponto
```

A qualidade do mapa acompanha a do OCR. Por isso o **OCR otimizado para documentos jurídicos
brasileiros** (carimbos, assinaturas digitais, varas e sistemas diversos) faz diferença: texto
fiel entra, mapa preciso sai. Gere o Markdown dos seus autos em **[ocr.tecjustica.com](https://ocr.tecjustica.com/)**.

---

## 1. Baixar

Pegue o arquivo do seu sistema na página de **[Releases](https://github.com/marcosmarf27/tj-mapa-cli/releases/latest)**:

| Sistema | Arquivo |
|---|---|
| **Windows** (x86-64) | `tj-mapa_windows_amd64.exe` |
| **Linux / WSL** (x86-64) | `tj-mapa_linux_amd64` |
| **Skill do Claude Code** | `tj-mapa.skill` |

## 2. Instalar (deixa `tj-mapa` disponível em qualquer terminal)

Rode **uma vez, pelo caminho do arquivo baixado** — ainda não precisa estar no PATH; é o
próprio `install` que o coloca lá:

**Windows (PowerShell):**
```powershell
.\tj-mapa_windows_amd64.exe install
```
**Linux / WSL:**
```bash
chmod +x ./tj-mapa_linux_amd64
./tj-mapa_linux_amd64 install
```

Abra um **novo terminal** e pronto — `tj-mapa` funciona em qualquer lugar. Instalação
**por usuário** (sem admin). Para remover: `tj-mapa uninstall`.

## 3. Configurar as duas chaves

Uma vez por máquina. Cada comando pede a chave e lê do teclado (não fica no histórico):

**1) OpenRouter — o motor de IA (faz o trabalho)**
```
tj-mapa config set        # cola a chave e tecla Enter
```
Pegue em **[openrouter.ai/keys](https://openrouter.ai/keys)** (começa com `sk-or-...`).

**2) TecJustiça — a sua licença (a mesma chave do OCR)**
```
tj-mapa config set-ocr    # cola a chave e tecla Enter
```
Pegue/gerencie em **[tecjustica-dashboard...railway.app](https://tecjustica-dashboard-production.up.railway.app)** (começa com `tjp_...`).

Confira quando quiser com `tj-mapa config show`.

> **Atalho:** dá pra configurar as duas já na instalação —
> `... install --key sk-or-... --ocr-key tjp_...`
>
> Também funcionam variáveis de ambiente (`OPENROUTER_API_KEY`, `TECJUSTICA_PARSE_KEY`) ou um
> arquivo `.env` no diretório. Quem só quer **conferir a leitura** dos autos sem IA pode rodar
> `tj-mapa -in processo.md -fase 0` — essa parte é offline e **não exige chave nenhuma**.

## 4. Usar

```
tj-mapa -in processo.md -formato ambos
```

Gera **`mapa.json`** (para o agente consumir) e **`mapa.md`** (tabela legível, com custo e tempo
no topo). A entrada `processo.md` é o Markdown vindo do OCR. Todas as opções: `tj-mapa -h`.

### ⚡ Mais rápido em processos grandes

Por padrão já é rápido. Em processos muito grandes dá pra acelerar com **`-concorrencia`** (faz
mais coisas ao mesmo tempo) e, quando você só quer a estrutura, **`-sem-enriquecer`** (pula a
parte mais demorada):

| Quero… | Comando |
|---|---|
| O mapa completo (padrão) | `tj-mapa -in processo.md -formato ambos` |
| Acelerar um processo grande | `tj-mapa -in processo.md -concorrencia 18` |
| Só a estrutura, o mais rápido | `tj-mapa -in processo.md -sem-enriquecer -concorrencia 24` |

Todas as opções: `tj-mapa -h`. Ajuste fino de desempenho no
**[README do projeto](https://github.com/marcosmarf27/tj-mapa#-velocidade--concorrência)**.

## 5. Skill do Claude Code (opcional)

Baixe `tj-mapa.skill` (é um zip) e extraia na pasta de skills do Claude Code:

```bash
# WSL / Linux
unzip tj-mapa.skill -d ~/.claude/skills/
```
```powershell
# Windows
Expand-Archive .\tj-mapa.skill -DestinationPath "$env:USERPROFILE\.claude\skills\"
```

Reinicie o Claude Code. A skill traz o binário embutido e dispara quando você menciona
**"tj-mapa"** ou pede o **"mapa do processo"**.

---

*Software proprietário da TecJustiça. Distribuído como binário compilado; o código-fonte não é público.*
