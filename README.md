# tj-mapa

**Mapa estrutural de um processo judicial inteiro — em segundos, por centavos.**

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

## 3. Configurar a chave (OpenRouter)

```
tj-mapa config set        # cola a chave e tecla Enter
```

Pegue a sua chave em [openrouter.ai/keys](https://openrouter.ai/keys). Também funciona a
variável de ambiente `OPENROUTER_API_KEY` ou um arquivo `.env`.

> Dá para instalar e já configurar a chave de uma vez: `... install --key sk-or-...`

## 4. Usar

```
tj-mapa -in processo.md -formato ambos
```

Gera **`mapa.json`** (para o agente consumir) e **`mapa.md`** (tabela legível, com custo e tempo
no topo). A entrada `processo.md` é o Markdown vindo do OCR. Todas as opções: `tj-mapa -h`.

### ⚡ Acelerar (concorrência)

O pipeline já paraleliza as chamadas de IA em lotes. A alavanca principal é **`-concorrencia`**
(workers paralelos, default `6` — conservador); quando você só quer o índice, **`-sem-enriquecer`**
é o maior ganho isolado (pula a fase que dominava ~70% do tempo e do custo).

| Cenário | Comando |
|---|---|
| Padrão (mapa completo) | `tj-mapa -in processo.md -formato ambos` |
| Processo grande (300–600 pág) | `tj-mapa -in processo.md -concorrencia 18` |
| Processo enorme (600+ pág) | `tj-mapa -in processo.md -concorrencia 24 -batch 30` |
| Só a estrutura, no talo | `tj-mapa -in processo.md -sem-enriquecer -concorrencia 24` |
| Turbo (completo) | `tj-mapa -in processo.md -formato ambos -concorrencia 24 -batch 30 -enriquecer-batch 12` |

Tempo de rede ≈ **O(segmentos / (`batch` · `concorrencia`))** — subir qualquer um dos dois corta o
tempo. Teto prático: ~64 conexões quentes; acima disso vale o **rate limit do provedor** (muitos
`429` → baixe a concorrência, o backoff já segura). A qualidade **não** muda com mais workers.
Referência completa de flags no **[README do projeto](https://github.com/marcosmarf27/tj-mapa#-velocidade--concorrência)**.

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
