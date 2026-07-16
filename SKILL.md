---
name: criar-carrossel-instagram
description: >
  Gera carrosseis para o Instagram — slides HTML deslizáveis com identidade
  visual do cliente + exportação individual em PNG 1080×1350px via Playwright.
  Use quando o usuário pedir carrossel, slides para Instagram, post de
  múltiplos slides ou conteúdo educativo em sequência.
---

# Carrossel Instagram

Gera carrosseis completos com a identidade visual do cliente aplicada: HTML
deslizável para aprovação + exportação em PNG 1080×1350px por slide.

Skills relacionadas: [[criar-post-instagram]] · [[validar-arte-visual]] · [[criar-legenda-instagram]] · [[classificar-icones]]

Os componentes visuais, regras de imagem, papéis de ícone, arquitetura de HTML e mecânica geral de exportação vêm de `_sistema/referencias/templates-post-instagram.md` (seções "Componentes exclusivos de carrossel" e "Família Carrossel" cobrem o que é específico de múltiplos slides). O schema de dados do brand-profile está em `_sistema/referencias/brand-profile-conteudo-visual.md`. Este arquivo documenta só o que é exclusivo desta skill: rubrica de escolha de hook, sequências de slides por tipo de conteúdo, humanização em lote e o script de exportação multi-slide.

---

## Pré-requisito 0 — Qual cliente

1. Se o usuário nomear o cliente/pasta explicitamente na mensagem → usar essa pasta como `{cliente}`.
2. Senão, procurar as pastas de topo do vault que contêm `brand-profile.md` na raiz:
   - Exatamente 1 encontrada → usar essa, sem perguntar.
   - 0 encontradas → perguntar ao usuário o nome do cliente/pasta (esperado: `{pasta}/brand-profile.md`).
   - 2+ encontradas → perguntar qual cliente usar antes de prosseguir.
3. Todos os paths e exemplos deste documento usam `{cliente}` como placeholder.

---

## Pré-requisito — Brand Profile

Antes de qualquer geração, ler `{cliente}/brand-profile.md` (schema completo em `_sistema/referencias/brand-profile-conteudo-visual.md`) e carregar:

- **Paleta de cores:** tokens `{brand_primary}`, `{brand_accent}`, `{brand_ink}`, `{brand_bg}`, `{brand_bg_alt}`, `{card_white}`, `{border}`, `{brand_primary_rgb}` (RGB sem parênteses, ex: `27,122,110`)
- **Tipografia e espaçamento:** ver seções "Tipografia" e "Espaçamento — Carrossel"
- **Vocabulário de ícones temáticos:** ver seção própria em brand-profile.md (mapeamento tema → ícone)
- **Regra de fundo:** ver "Regra de fundo" — cada cliente define a própria (não assumir fundo claro como universal)
- **Logo:** posição, largura, caminho → `{path_logo}`
- **Frame Instagram:** `{brand_handle}`, `{brand_profile_name}`
- **Vocabulário do público:** espelho de linguagem para slides 2–N e CTA
- **Humanização:** tabela de padrões AI → substitutos humanos
- **Princípios de design e restrições visuais:** guiam composição e revisão
- **Dados da empresa:** endereço, telefone, WhatsApp, horário, serviços e diferenciais — sempre que um slide citar um dado factual, usar exatamente o que está nessa seção. Nunca inventar, aproximar ou supor serviço, estrutura ou informação que não conste lá; se faltar o dado, perguntar ao usuário
- **Caminhos:** `{path_html_temp}` (carousel.html), `{path_output}`, `{path_output_abs}`, `{vault}`

---

## Passo 1 — Coletar dados do post

Perguntar ao usuário antes de gerar:

1. **Tipo de conteúdo** — Educativo · Depoimento · Dicas/Listicle · Marco Numérico · Data Comemorativa
2. **Tema ou assunto** — o que o carrossel vai ensinar, mostrar ou comunicar
3. **Imagens disponíveis** — fotos reais, objetos, ícones, ou gerar só com texto/elementos gráficos
4. **Número de slides** — sugestão automática por tipo (ver sequências abaixo)
5. **CTA final** — WhatsApp, Direct, contato, ou personalizado

Se o usuário disser "faz um carrossel sobre X" sem mais detalhes, inferir o tipo
mais adequado ao tema e confirmar antes de gerar.

---

## Passo 1.1 — Criar checklist de execução (obrigatório)

Antes de gerar as opções de capa, gerar um identificador curto pra esta execução (ex: horário atual) e invocar `/auditoria-execucao` em modo bootstrap, passando `criar-carrossel-instagram:<id-execucao>` e os gates: Passo 1.8 (Humanizar todos os slides), Passo 1.9 (Verificação visual — 1 item por slide, ex: "Validar slide 1/{N}"..."Validar slide {N}/{N}", mais "Validação narrativa do conjunto" e "Checklist específico de carrossel"), Passo 1.10 (Criar legenda). Guardar esse `<id-execucao>` pra usar também no fechamento.

**Por padrão, para cada slide, antes de perguntar ao usuário se há foto própria ou decidir por um slide 100% tipográfico (ícone + texto), tentar o banco de imagens primeiro** — mesma lógica de `criar-post-instagram`: chamar `buscar_imagem(cliente: "{cliente}", consulta: "<tema do slide>")` via MCP `banco-imagens` (ver [[mcp-banco-imagens]]).

**Score alto não é sinônimo de coerência temática — checar os dois antes de sugerir.** `score` ≥ 0.65 mede similaridade textual da consulta, não garante que a cena bate com a emoção/situação específica do slide. Ex: buscar por "pânico, travar na prova" pode retornar cenas de condução tranquila e confiante (score 0.65–0.67), tecnicamente próximas mas tematicamente erradas para um slide sobre nervosismo/medo. Avaliar a imagem em si contra o tema real do slide antes de sugerir; se não bater, não sugerir só porque o score passou do threshold.

Quando a imagem bater em score e coerência, sugerir ao usuário antes de embutir; só usa após confirmação. Uma foto real (mesmo institucional, tipo fachada) costuma elevar o nível da capa/slide bem mais que ícone puro.

**Antes de desistir e cair pro tipográfico, tentar reformular a busca** (outra consulta, outra categoria de `listar_indice` plausível pro tema) em vez de decidir sozinho na primeira tentativa sem sucesso — uma categoria mais específica pode ter exatamente a foto certa mesmo quando a consulta genérica não achou nada coerente. **Só cai para o fluxo manual (perguntar foto própria ao usuário, ou seguir com ícone puro) depois de tentar reformular e mesmo assim não haver imagem com contexto relevante** — banco vazio, todas já usadas, nenhuma reformulação resolveu, ou MCP indisponível. Ao cair pro fluxo manual, informar rapidamente ao usuário o que foi tentado, pra que ele possa apontar uma categoria/pasta específica se souber de uma. Nunca bloquear a geração por causa disso.

**Foto de produto/objeto com proporção muito diferente do slide:** não forçar `cover`/`contain` num card pequeno até parecer certo por tentativa — ver Regra 6 de "Imagens fornecidas pelo usuário" e a variação "Foto de topo (full-bleed, sem sobreposição)" do Layout E2 em `templates-post-instagram.md`. Se o assunto precisa ficar 100% visível, um slide inteiro dedicado à foto (sem texto sobreposto) costuma resolver melhor que espremer a foto atrás do texto de um slide de conteúdo.

---

## Passo 1.3 — Gerar opções de capa (Slide 1)

Invocar `/criador-de-hooks-para-carrossel-no-instagram` com os dados coletados no Passo 1,
pré-preenchendo o contexto fixo (valores de `{cliente}/brand-profile.md` → Contexto do cliente):

```
Tema do carrossel: [tema do Passo 1]
Nicho: {nicho}
Público-alvo: {publico_alvo}
Objetivo do carrossel: [educativo / engajamento / autoridade — conforme Passo 1]
Tom: {tom}
```

**Filtro de marca — aplicar antes de apresentar ao usuário:**
- Eliminar opções com tom agressivo, negativo ou que gerem insegurança
- Eliminar opções que contrariem a Regra de fundo do brand-profile do cliente
- Priorizar: educativo, número/lista, pergunta direta

**Gerar 4 opções internamente (não apresentar ao usuário ainda):**
- 1 educativa ("O que ninguém te contou sobre…")
- 1 com número/lista ("5 erros que fazem você perder tempo com…")
- 1 com pergunta direta ("Você sabe o que realmente pesa aqui?")
- 1 emocional (frase de impacto sem estrutura de lista)

Avançar imediatamente para o Passo 1.4.

---

## Passo 1.4 — Eleger o hook campeão (rubrica autônoma)

Aplicar a rubrica abaixo nas 4 opções geradas no Passo 1.3. **Não perguntar ao usuário — fazer a escolha de forma autônoma e apresentar apenas o resultado.**

### Rubrica — 6 critérios, nota 1–3 por critério

| Critério | Peso | O que avaliar |
|---|---|---|
| Tensão de informação | ×3 | Abre uma lacuna que só fecha deslizando? Sem isso não há razão mecânica para swipe. "Você está errando isso." cria tensão. "Dicas sobre X" não cria nada. |
| Especificidade | ×2 | Quanto mais específico, mais crível e mais curioso. Número + consequência = especificidade real. "5 erros que custam caro" > "Erros comuns". |
| Identificação imediata | ×2 | O leitor lê e pensa "isso é sobre mim"? Deve nomear a dor ou o perfil sem esforço de interpretação. |
| Força do gatilho | ×2 | O gatilho é o certo para o objetivo? Vender → medo de perder. Engajar → identificação. Autoridade → curiosidade + status. Gatilho errado = hook fraco mesmo bem escrito. |
| Economia de palavras | ×1 | Dentro de 12 palavras, cada palavra carrega peso? 8 palavras fortes batem 12 medianas. Subtexto que repete o principal é desperdício. |
| Promessa cumprível | ×1 | O carrossel entrega o que o hook promete? Inflação de expectativa reduz salvamentos e aumenta unfollows. |

**Pontuação máxima:** 33 pontos (3×3 + 3×2 + 3×2 + 3×2 + 3×1 + 3×1)

**Desempate:** empate na pontuação total → vence o com nota mais alta em Tensão de informação. É o único critério mecânico — os outros são de qualidade, esse é de sobrevivência no feed.

### Formato de saída

Apresentar ao usuário apenas:
1. Tabela de pontuação completa (para transparência)
2. Hook campeão destacado com justificativa em 1 linha
3. Avançar para o Passo 1.5 sem aguardar aprovação

```
| Critério         | Peso | Hook A | Hook B | Hook C | Hook D |
|------------------|------|--------|--------|--------|--------|
| Tensão           | ×3   |        |        |        |        |
| Especificidade   | ×2   |        |        |        |        |
| Identificação    | ×2   |        |        |        |        |
| Força do gatilho | ×2   |        |        |        |        |
| Economia         | ×1   |        |        |        |        |
| Promessa         | ×1   |        |        |        |        |
| **Total**        |      |        |        |        |        |

🏆 Hook campeão: [Hook X] — [justificativa em 1 linha]
```

---

## Passo 1.5 — Ícone temático como elemento de arte

O ícone (PNG próprio do Banco de Ícones, ou Font Awesome 6 como fallback) é um **elemento visual da arte**, não apenas um rótulo. Pode ser aplicado em **todos os tipos de carrossel e em qualquer slide** — capa, conteúdo ou CTA.

Antes de escolher o ícone, invocar `classificar-icones` — se houver PNG pendente em `{cliente}/Identidade Visual/Ícones/_entrada/`, ele é classificado e nomeado ali, ficando disponível pra esta escolha; se a pasta estiver vazia, a skill não faz nada e segue o fluxo normal.

Vocabulário (qual ícone usar por tema, PNG próprio ou Font Awesome) e a técnica de CSS mask pra recolorir o PNG: ver `{cliente}/brand-profile.md` → "Vocabulário de ícones temáticos".

Mecânica de aplicação (papéis — hero, badge, fundo fantasma, decorativo, rótulo — com tamanho, posição e cor por papel): ver `_sistema/referencias/templates-post-instagram.md` → "Papéis do ícone temático".

**Nuance de carrossel:** fundo fantasma pode ser somado a qualquer papel e é ótimo para dar textura nos slides de conteúdo (2–N) sem interferir no texto, mantendo o mesmo ícone entre slides pra dar continuidade visual à sequência.

**Capa com foto emocional/expressiva — nascer já com impacto, não esperar o usuário pedir.** Quando o Slide 1 usar uma foto de pessoa com expressão forte (nervosismo, torcida, susto, alívio), a primeira versão já deve maximizar o peso visual: painel de foto ocupando a maior área possível dentro do slide (não um recorte tímido), título em fonte grande (próxima do teto seguro de largura, ver fórmula de linha órfã em `templates-post-instagram.md`), e a palavra de maior tensão do hook em `{brand_primary}` dentro do título. Uma capa "correta" mas morna (foto pequena, título de tamanho médio, sem destaque de cor) tende a precisar do mesmo ajuste depois — fazer isso na primeira versão evita uma rodada de revisão previsível.

---

## Passo 1.8 — Humanizar TODOS os slides (gate obrigatório)

**NUNCA gerar o HTML sem antes invocar formalmente o `/humanizer` com o texto de todos os slides.**
Isso inclui: capa (Slide 1), slides de conteúdo (2–N) e slide de CTA final.

**Escopo do input ao humanizer — incluir:**
- Slides 2–N: título do slide + corpo + itens de lista (se houver)
- Slide final (CTA): frase emocional + chamada para ação

**Slide 1 — excluído desta etapa:** o texto da capa vem do `/criador-de-hooks-para-carrossel-no-instagram` (Passo 1.3), que já produz copy humana por construção. Não passar o Slide 1 pelo humanizer.

**Como executar:**

1. Redigir rascunho completo de todos os slides em texto plano (sem HTML)
2. Invocar `/humanizer` passando o rascunho inteiro como input — nunca slide por slide
3. Usar o texto final retornado no HTML — nunca o rascunho direto
4. Se o usuário questionar se o humanizer foi aplicado → significa que não foi percebido; rever o texto e reaplicar

**Referência rápida de padrões a corrigir:** ver tabela em `{cliente}/brand-profile.md` → seção **Humanização**.

**Critério de aprovação:** qualquer frase do carrossel passa se alguém poderia ter escrito assim numa mensagem de WhatsApp.

**Atenção especial aos títulos dos slides (2–N), não só ao corpo:** o humanizer costuma revisar bem os parágrafos de corpo mas deixar passar título fraco ou genérico, porque título curto não aciona os mesmos padrões de detecção. Antes de aceitar o retorno do humanizer, checar cada título isoladamente contra estes dois problemas:
- **Negação dupla/paralela** ("Não é só X, não é só Y", "Não é sobre X, é sobre Y") — quase sempre soa robótico e não diz nada concreto. Reescrever afirmando o que o slide de fato mostra
- **Vago/genérico** (frases que serviriam pra qualquer carrossel de qualquer nicho, tipo "Pensado pra cada situação", "Feito especialmente pra você") — o teste: se o título ficaria igual de verdadeiro trocando o assunto do slide por outro qualquer, é genérico demais. Nomear a audiência ou a situação específica do slide funciona bem

---

## Passo 1.9 — Verificação visual (gate obrigatório antes de apresentar ao usuário)

**NUNCA apresentar o carrossel ao usuário sem antes verificar todos os slides visualmente.** Esta é a autoavaliação visual desta skill — o equivalente multi-slide do Passo 2.6 de `criar-post-instagram`: cada slide é renderizado na resolução final e passa por `/validar-arte-visual` (ver [[validar-arte-visual]]), a mesma ferramenta usada no post único. A diferença é só de escala — aqui há N imagens pra validar em sequência, não uma.

Problemas encontrados devem ser corrigidos silenciosamente — o usuário recebe o resultado final, não o processo de debug.

### Como executar

**Renderizar cada slide já na resolução final (1080×1350, mesma regra do Fluxo de revisão — nunca validar sobre screenshot em resolução de viewport 1x).** Reaproveitar exatamente o mecanismo da seção "Script de exportação (loop multi-slide)" mais abaixo, mudando só o destino: `OUTPUT_DIR` apontando para `{cliente}/_tmp/` em vez da pasta de saída final, e os nomes salvos como `slide_XX_check.png`. Isso evita manter dois métodos de captura divergentes (um de baixa resolução só pra QA, outro de alta resolução pra exportação) — o mesmo script serve às duas finalidades, só muda o destino e o momento em que roda.

**Regra de path:** sempre usar `{cliente}/_tmp/<nome>.png`. O CWD do MCP Playwright é a raiz do vault — filenames sem prefixo de pasta criam sujeira na raiz.

### Autoavaliação por slide — `/validar-arte-visual`

Para cada slide `i` (1 a N), na ordem, com o PNG já renderizado em resolução final:

**Antes de invocar `/validar-arte-visual`, se o slide usar algum elemento posicionado de forma absoluta/flutuante perto de outro bloco (badge, pill, selo, "Arrasta para ver", ou texto de CTA/corpo próximo a um logo posicionado em `position:absolute` no rodapé, especialmente no slide final):** medir as caixas delimitadoras dos elementos envolvidos (via `getBoundingClientRect` no browser, comparando `top`/`bottom`/`left`/`right` de cada um) e confirmar que não há sobreposição não intencional antes de considerar o slide pronto pra autoavaliação. Não confiar só na leitura visual do preview — uma sobreposição de poucos pixels no preview vira uma faixa visível na exportação final em escala maior.

**Essa medição não é condicional a "parecer arriscado":** sempre que o corpo/título do slide tiver comprimento variável e estiver no mesmo bloco visual que um CTA, medir é obrigatório — ver "Quando medir sobreposição deixa de ser opcional" em `templates-post-instagram.md`. Checar também linha viúva na última linha de qualquer texto multi-linha do slide, e se o CTA do slide final está no mesmo flex container do ícone/título/corpo (não num `position:absolute` separado) — ver "Quando o CTA é parte do bloco de conteúdo" no mesmo arquivo.

Primeiro rodar a "Verificação automática de contraste (medição por pixel)" de `_sistema/referencias/templates-post-instagram.md` sobre o slide — corrigir qualquer bloco de texto reprovado antes de seguir. Só depois invocar `/validar-arte-visual` passando:
- O caminho de `{cliente}/_tmp/slide_XX_check.png` do slide `i`
- O contexto de marca (paleta e princípios do brand-profile)
- Canal: `slide {i} de {N} de um carrossel Instagram`, informando o papel do slide na sequência (capa / conteúdo / CTA final, conforme a tabela do Passo 2)
- **Elementos obrigatórios: logo, se essa for a regra fixa da marca do cliente** (mesma checagem de `criar-post-instagram`)
- **CTA obrigatório só no slide final** — os demais slides (capa e conteúdo) não são penalizados pela ausência de CTA

Por slide:
- Veredito "Aprovada" ou "Aprovada com observações" → seguir para o próximo slide (repassar as observações ao usuário ao final, se houver)
- Veredito "Solicitar ajustes antes de aprovar" ou "Reprovada" → corrigir os itens de prioridade alta/média apontados no relatório **só daquele slide**, re-renderizar apenas ele e repetir a validação antes de seguir para os próximos — não é necessário re-exportar o carrossel inteiro

**Teto de segurança: 7 tentativas por slide.** Se depois de 7 rodadas de correção naquele slide o veredito ainda não for "Aprovada" ou "Aprovada com observações", parar o loop **daquele slide**, seguir para os demais, e apresentar o carrossel inteiro ao usuário ao final — com o relatório da última autoavaliação do(s) slide(s) que não convergiram e uma nota explícita do que não foi possível resolver sozinho. Nunca insistir indefinidamente sem informar o usuário.

Usar o "Guia de correção rápida" de `criar-post-instagram` (Passo 2.6) como referência de ajuste técnico — os mesmos problemas (respiro concentrado, sombra pesada, CTA/logo desproporcionais, texto de CTA sobrepondo logo no rodapé do slide final, linha viúva, CTA fora do bloco de conteúdo, corte de rosto, elemento decorativo fraco, dado suspeito, elemento perto da borda) se aplicam ao HTML de slide de carrossel do mesmo jeito.

### Validação narrativa do conjunto (além dos slides individuais)

Depois que todos os slides passarem individualmente pela autoavaliação acima, checar o carrossel como sequência, não só slide a slide — ver "Validação narrativa do carrossel" em `_sistema/referencias/templates-post-instagram.md` (promessa da capa cumprida, progressão lógica, texto equilibrado entre slides, fechamento retomando a capa, CTA no slide certo).

### Critérios de aprovação específicos de carrossel (complementares a `/validar-arte-visual`)

Os 12 critérios de `/validar-arte-visual` avaliam cada slide como peça única — não cobrem o que é específico de sequência/carrossel. Verificar também estes pontos, todos obrigatórios:

| Critério | Verificar |
|---|---|
| Nenhum elemento cortado | Texto, logo, botão e ícone completamente dentro do slide |
| Nenhuma sobreposição | Texto não sobrepõe logo, botão não sobrepõe logo |
| Badge redondo | `border-radius:50%` + `min-width` e `min-height` definidos (sem esses, flex estica o badge em oval) |
| "Arrasta para ver" visível | Deve aparecer na capa sem sobreposição com subtitle — usar `position:absolute;bottom:Xpx` se necessário |
| Fundo consistente com a regra do cliente nos slides 1 a N-1 | Nenhum slide de conteúdo contraria a Regra de fundo do brand-profile |
| Card/caixa de destaque inteiro | Card de prova social ou caixa de dados visível e não cortado na base |
| Logo no CTA | No slide gradiente, logo deve aparecer abaixo do botão (não atrás) — usar `bottom:6px` se houver conflito |
| Conteúdo não transbordando | Verificar que `justify-content:flex-start` + `padding-bottom` reservam espaço suficiente para todos os elementos |

Só depois de: autoavaliação por slide aprovada (ou aprovada com observações) + validação narrativa do conjunto + checklist específico acima — apagar `{cliente}/_tmp/` e prosseguir para o Passo 2:

```python
import shutil
from pathlib import Path
tmp = Path("{vault}/{cliente}/_tmp")
if tmp.exists():
    shutil.rmtree(tmp)
```

### Ajuste de escala para o preview (420×525)

Os tamanhos de fonte da SKILL foram definidos para exportação 1080×1350. No preview 420×525, textos longos quebram mais linhas. Regras de ajuste:

| Fonte declarada na SKILL | Tamanho seguro no preview |
|---|---|
| 60–80px (título principal) | 42–50px |
| 44–56px (subtítulo) | 32–40px |
| 32–40px (corpo) | 22–26px |
| 80–120px (script principal) | 52–62px |
| 40–60px (script secundário) | 30–40px |

Se um elemento transbordar após aplicar esses tamanhos, reduzir mais ou encurtar o texto — nunca deixar conteúdo cortado.

---

## Passo 1.10 — Criar legenda (depois de todos os slides validados)

Só depois que o Passo 1.9 aprovar todos os slides (autoavaliação individual + validação narrativa do conjunto + checklist específico), invocar `/criar-legenda-instagram` (ver [[criar-legenda-instagram]]) passando:

- **Formato:** carrossel ({N} slides)
- **Ângulo:** o Tipo de conteúdo definido no Passo 1 — mapeia direto pra tabela de ângulos de `/criar-legenda-instagram` (Educativo → Educativo; Depoimento → Depoimento/prova social; Dicas/Listicle → Educativo; Marco Numérico → Marco numérico; Data Comemorativa → Institucional/data comemorativa), sem tradução extra necessária
- **O que de fato ficou na arte final:** hook vencedor da capa (Passo 1.4), título/texto de cada slide de conteúdo e o CTA do slide final — já humanizados (Passo 1.8) e validados (Passo 1.9)
- **Contexto de marca:** brand-profile (Pré-requisito)
- **CTA desejado:** o CTA final coletado no Passo 1; se nada tiver sido especificado, o CTA padrão do brand-profile

O bloco de legenda + hashtags retornado é apresentado junto com o preview no Fluxo de revisão (abaixo) — nunca só os slides sozinhos.

---

## Passo 2 — Sequências de slides por tipo

Ver `{cliente}/brand-profile.md` para valores exatos dos tokens de cor de cada fundo. Papéis de slide genéricos (Capa, Desenvolvimento, Exemplo, Checklist, Fechamento/CTA) em `_sistema/referencias/templates-post-instagram.md` → "Família Carrossel"; as sequências abaixo são a aplicação concreta desses papéis, com exemplos ilustrativos de conteúdo.

**Fechamento pode reaproveitar a foto da capa como fundo (bookending):** quando a capa usa uma foto real (banco de imagens) em card, o slide de Fechamento/CTA pode reaproveitar a mesma foto como fundo cheio, com overlay do gradiente da marca por cima (`linear-gradient` nas cores de `{brand_primary}`/`{brand_accent}` a ~85–90% de opacidade sobre a foto, texto branco por cima). Fecha o carrossel visualmente retomando a capa, em vez de um gradiente genérico solto. Só funciona quando a foto é boa o suficiente pra aguentar esse peso duas vezes — não forçar se a única foto disponível for fraca.

### Educativo (5–7 slides) — baseado no Template D

| # | Tipo | Fundo | Conteúdo | Ícone |
|---|---|---|---|---|
| 1 | Capa | {brand_bg} | Título forte + "Arrasta →" | Hero, Badge ou Decorativo em linha |
| 2–N | Conteúdo | Alternando {brand_bg} / {brand_bg_alt} | Título do ponto + texto + caixa de destaque opcional | Fundo fantasma (sutil) |
| Final | CTA | Gradiente de marca | Frase emocional (fonte de destaque) + CTA + contato | Badge branco ou fundo fantasma branco |

### Depoimento expandido (3–5 slides)

| # | Tipo | Fundo | Conteúdo | Ícone |
|---|---|---|---|---|
| 1 | Gancho | {brand_bg} | Aspas abertura (fonte de destaque) + início do depoimento | Fundo fantasma temático (ex: `fa-star`, `fa-heart`) |
| 2–N | Continuação | {brand_bg_alt} | Continuação do depoimento | Fundo fantasma (mantém o mesmo ícone) |
| Final | Identidade | {brand_bg} | Nome do protagonista + ⭐⭐⭐⭐⭐ + CTA sutil | Badge ou fundo fantasma |

### Listicle — dicas, erros, fatos (5–8 slides)

| # | Tipo | Fundo | Conteúdo | Ícone |
|---|---|---|---|---|
| 1 | Hook | {brand_bg} | Número em destaque + promessa | Hero ou Badge |
| 2–N | Item N | Alternando | Número do item + título + explicação curta | Fundo fantasma (o mesmo ícone da capa) |
| Final | CTA | Gradiente de marca | Síntese + CTA | Fundo fantasma branco |

### Marco Numérico (3–4 slides)

| # | Tipo | Fundo | Conteúdo | Ícone |
|---|---|---|---|---|
| 1 | O número | {brand_bg} | Número gigante + linha + descrição | Fundo fantasma centralizado (ex: `fa-trophy`) |
| 2–N | Contexto | {brand_bg_alt} | O que esse número representa + prova social | Fundo fantasma (continuidade visual) |
| Final | CTA | Gradiente de marca | Convite + contato | Badge branco |

### Data Comemorativa (3–5 slides)

| # | Tipo | Fundo | Conteúdo | Ícone |
|---|---|---|---|---|
| 1 | Tema | {brand_bg} | Data/tema + linha emocional (fonte de destaque) | Hero ou Badge — ícone da data |
| 2–N | Conexão | {brand_bg_alt} | Conexão emocional com o benefício central do produto/serviço (ver "Contexto do cliente" no brand-profile) | Fundo fantasma |
| Final | CTA | Gradiente de marca | Mensagem de fechamento + contato | Fundo fantasma branco |

---

## Passo 3 — Imagens, arquitetura e componentes

Seguir `_sistema/referencias/templates-post-instagram.md` para:

- Regras de imagem fornecida pelo usuário (base64, nunca caminho relativo, gerar via Python)
- Formato do post-slide (perfil Feed 4:5: 420×525 viewport, 1080×1350 exportação)
- Logo (posição conforme brand-profile, obrigatório ou não conforme regra do cliente)
- Componentes reutilizáveis (aspas, faixas, caixas de destaque, ícones, CTA)
- Componentes exclusivos de carrossel (seta de swipe, "Arrasta para o lado", número do item, dots)
- Frame Instagram de preview (com dots, já que é carrossel)

Overlay para foto de fundo em slide claro: `rgba({brand_bg_rgb},0.35)` (mesma opacidade de `templates-post-instagram.md` → "Imagens fornecidas pelo usuário"; componentes RGB de `{cliente}/brand-profile.md`).

---

## Fluxo de revisão

**Auditoria antes de apresentar (obrigatório):** invocar `/auditoria-execucao` em modo fechamento, passando `criar-carrossel-instagram:<id-execucao>` (o mesmo gerado no bootstrap). Se retornar `AUDITORIA_INCOMPLETA` (inclusive por falta de validação de algum slide), validar o slide faltante e repetir a auditoria antes de mostrar qualquer preview.

Seguir o "Fluxo de revisão — loop obrigatório" de `_sistema/referencias/templates-post-instagram.md` (incluindo a regra de preview sempre em resolução final via `device_scale_factor`). Apresentar a legenda do Passo 1.10 junto com o preview dos slides.

**Adendo específico desta skill:** ao receber feedback, corrigir apenas os slides mencionados, não regenerar o carrossel inteiro. Se a correção mudar texto ou dado que a legenda referencia, repetir o Passo 1.10 antes de exportar.

---

## Exportação — PNG 1080×1350px por slide

A mecânica geral (viewport, `device_scale_factor`, por que funciona, erros comuns) está em `_sistema/referencias/templates-post-instagram.md` → "Exportação — perfil Feed 4:5". Esta seção cobre só o que é específico de exportar **múltiplos slides de um carrossel**, não um post único.

### Pasta de saída padrão

Salvar sempre em `{path_output}<AAAA-MM-DD>-<tema>/`:
- Criar a subpasta com data primeiro, depois o tema — facilita ordenação cronológica
- Exemplo: `{cliente}/Posts/2026-07/Postagens/2026-07-01-tema/`
- Nomear os arquivos com zero à esquerda: `slide_01.png`, `slide_02.png`, etc. (evita ordem quebrada em listagens se o carrossel um dia passar de 9 slides)
- Salvar também a legenda final do Passo 1.10 em `legenda.md`, na mesma pasta dos slides, logo após a exportação bem-sucedida — nunca antes de o carrossel ser de fato exportado. Formato:

```markdown
---
data: <AAAA-MM-DD>
tema: <tema>
tipo: <Tipo de conteúdo do Passo 1>
slides: <N>
status: pronto
---

# Legenda — <tema>

<texto da legenda>

---

**Hashtags:**
<hashtags>
```

O HTML intermediário fica em `{path_html_temp}`.

### Arquivos temporários — regra obrigatória (sem sujeira na raiz)

Se algum passo usar o MCP Playwright (em vez do script Python de "Script de exportação (loop multi-slide)" abaixo) pra capturar screenshot — por exemplo, uma inspeção pontual fora do fluxo padrão — lembrar que ele salva relativo ao CWD = raiz do vault. Para evitar poluição:

**Regra:** sempre usar o prefixo `{cliente}/_tmp/` nos filenames do MCP Playwright.

```
# Correto:
filename: "{cliente}/_tmp/slide_01.png"

# ERRADO — vai para a raiz do vault:
filename: "slide-1.png"
```

**A exportação oficial não passa por `_tmp/` nem por reamostragem.** O "Script de exportação (loop multi-slide)" abaixo grava direto em `{path_output}<AAAA-MM-DD>-<tema>/`, já na resolução final (1080×1350) via `device_scale_factor` — essa regra de prefixo só vale pra usos pontuais do MCP Playwright, como o do Passo 1.9.

### Script de exportação (loop multi-slide)

**Reaproveitado também no Passo 1.9** (autoavaliação por slide via `/validar-arte-visual`): mesma lógica, só muda `OUTPUT_DIR` para `{cliente}/_tmp/` e o padrão de nome para `slide_XX_check.png`. Roda duas vezes no fluxo total — uma cedo, pra QA (apagado depois), outra aqui, pra exportação oficial após aprovação do usuário.

```python
import asyncio
from pathlib import Path
from playwright.async_api import async_playwright

INPUT_HTML = Path("{path_html_temp}")
OUTPUT_DIR = Path("{path_output_abs}<AAAA-MM-DD>-<tema>")
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

TOTAL_SLIDES = 7  # atualizar conforme o carrossel

VIEW_W = 420
VIEW_H = 525
SCALE = 1080 / 420  # 2.5714...

async def export_slides():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(
            viewport={"width": VIEW_W, "height": VIEW_H},
            device_scale_factor=SCALE,
        )

        html_content = INPUT_HTML.read_text(encoding="utf-8")
        await page.set_content(html_content, wait_until="networkidle")
        await page.wait_for_timeout(3000)  # aguardar Google Fonts

        await page.evaluate("""() => {
            document.querySelectorAll('.ig-header,.ig-dots,.ig-actions,.ig-caption')
                .forEach(el => el.style.display='none');

            // Propriedades individuais, nunca style.cssText = '...': cssText apaga todo
            // o estilo inline existente, inclusive o background da marca (bug real
            // encontrado em produção: post exportava com o cinza da página de preview
            // em vez do SLIDE_BG correto).
            const frame = document.querySelector('.ig-frame');
            Object.assign(frame.style, { maxWidth: 'none', borderRadius: '0', boxShadow: 'none', overflow: 'hidden', margin: '0' });

            const viewport = document.querySelector('.carousel-viewport');
            Object.assign(viewport.style, { aspectRatio: 'unset', overflow: 'hidden', cursor: 'default' });

            Object.assign(document.body.style, { padding: '0', margin: '0', display: 'block', overflow: 'hidden' });
        }""")
        await page.wait_for_timeout(500)

        for i in range(TOTAL_SLIDES):
            await page.evaluate("""(idx) => {
                const track = document.querySelector('.carousel-track');
                track.style.transition = 'none';
                track.style.transform = 'translateX(' + (-idx * 420) + 'px)';
            }""", i)
            await page.wait_for_timeout(400)

            await page.screenshot(
                path=str(OUTPUT_DIR / f"slide_{i+1:02d}.png"),
                clip={"x": 0, "y": 0, "width": VIEW_W, "height": VIEW_H}
            )
            print(f"Exportado slide {i+1}/{TOTAL_SLIDES}")

        await browser.close()

asyncio.run(export_slides())
```

### Erros específicos de carrossel a evitar

| Erro | Consequência | Correção |
|---|---|---|
| Não ocultar `.ig-dots` no export | Export inclui os dots de navegação | Ocultar junto com `.ig-header,.ig-actions,.ig-caption` |
| `transition` não removida antes do loop | Slides deslizam animando durante a captura | Sempre `transition:none` antes de trocar o `transform` |
| Não limpar `{cliente}/_tmp/` | Sujeira acumula no vault | `shutil.rmtree(TMP)` obrigatório ao final |
| Deixar cor de variável como nome | Cor inválida no CSS | Sempre interpolar o valor hex real no HTML gerado |

---

## Referências de design

Ver `{cliente}/brand-profile.md`:
- **Princípios de design** — princípios que guiam composição e escolha de layout
- **Restrições visuais** — checklist do que nunca fazer
- **Vocabulário de ícones temáticos** — mapeamento tema → ícone
