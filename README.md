# tj-mapa

**Índice navegável de processos judiciais para o seu agente de IA.**

Parte do ecossistema **TecJustiça**: OCR jurídico ([ocr.tecjustica.com](https://ocr.tecjustica.com/))
→ **tj-mapa** → MCP ([mcp.tecjustica.com](https://mcp.tecjustica.com/)) — usados em conjunto com o Claude Code.

---

## O que o tj-mapa faz

Quando um agente de IA precisa trabalhar um processo de 300, 500, 800 páginas, ele acaba
**relendo os autos inteiros** a cada pergunta — lento, caro em tokens e sujeito a erro. É como
reler um livro de 800 páginas toda vez que você quer achar um capítulo.

O `tj-mapa` resolve isso gerando o **mapa estrutural** do processo: ele localiza **cada peça**
(petição, contestação, despacho, decisão, sentença, certidão, ofício, mandado…) e devolve, para
cada uma:

- **Onde está** — intervalo exato de **página** e de **linha**;
- **O que é** — o tipo provável da peça;
- **Do que trata** — um resumo de uma ou duas frases;
- **A prova** — uma citação literal que ancora o trecho no documento original.

Não é um resumo do processo (resumir descarta o original e arrisca alucinação). É um **índice
com endereços** — o original continua sendo a fonte da verdade; o mapa só o torna **navegável**.

## O mapa na prática

```text
$ tj-mapa -in processo.md -formato ambos

  ✓ 312 páginas  →  47 peças mapeadas  ·  1m54s  ·  custo: US$ 0,04
  → mapa.json  /  mapa.md
```

Um trecho do `mapa.md` gerado:

| # | Páginas | Linhas | Peça | Resumo |
|---|--------|--------|------|--------|
| 2 | 2–5 | 46–277 | PETIÇÃO INICIAL | Denúncia do MP por furto qualificado. |
| 4 | 8–9 | 412–489 | DECISÃO | Recebimento da denúncia; designa audiência. |
| 23 | 142–148 | 13.820–14.310 | SENTENÇA | Condena o réu; pena em regime aberto. |

**O ganho para o agente de IA:**

> **Antes:** lê ~28.000 linhas para localizar a sentença.
> **Depois:** consulta o mapa, vê *"SENTENÇA — págs 142–148, linhas 13.820–14.310"* e lê **490 linhas**.

Mais rápido, mais barato e mais **preciso** — a IA trabalha sempre sobre o texto real, na
localização exata, e não sobre um resumo de segunda mão. Um processo inteiro é mapeado por
**centavos**, em poucos minutos.

## O diferencial: a inteligência está no algoritmo, não no LLM

Mandar o processo inteiro para um modelo de IA e pedir o índice seria **lento, caro e instável**
— o modelo alucina páginas e linhas, e cada nova pergunta recomeça do zero. O `tj-mapa` faz o
oposto: o trabalho pesado é de um **algoritmo proprietário da TecJustiça**, determinístico e
rodando localmente, que aciona o modelo **apenas no ponto exato que exige julgamento**, com
contexto mínimo. É daí que vem a combinação que parece contraditória — **rápido, custo de
centavos e preciso, tudo ao mesmo tempo**.

Ou seja: o segredo do mapeamento está na **engenharia do nosso algoritmo**, não em "jogar tudo
num LLM e torcer". O modelo é uma peça barata e substituível; a inteligência que coordena o
processo é o nosso diferencial — e o que torna isso viável na rotina, em escala e a baixo custo.

## De onde vem a entrada: o OCR jurídico da TecJustiça

O `tj-mapa` **não lê PDF**. A entrada dele é o **Markdown gerado pelo OCR jurídico da
TecJustiça** — é uma **dependência**: primeiro o PDF dos autos passa pelo OCR, que devolve texto
limpo com **uma página por marcador `## Página N`**; é desse formato que o `tj-mapa` parte.

```
   PDF dos autos
        │   OCR jurídico  →  ocr.tecjustica.com
        ▼
   Markdown  (uma página por "## Página N")
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
variável de ambiente `OPENROUTER_API_KEY` ou um arquivo `.env` no diretório de trabalho.

> Dá para instalar e já configurar a chave de uma vez: `... install --key sk-or-...`

## 4. Usar

```
tj-mapa -in processo.md -formato ambos
```

Gera **`mapa.json`** (para o agente consumir) e **`mapa.md`** (tabela legível, com custo e tempo
no topo). A entrada `processo.md` é o Markdown vindo do OCR (uma página por `## Página N`). Para
só a estrutura, mais rápido: adicione `-sem-enriquecer`. Todas as opções: `tj-mapa -h`.

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
