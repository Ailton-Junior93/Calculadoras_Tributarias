# Documentação Técnica — Calculadoras Tributárias

**Versão:** 4.5**Formato:** arquivo único `CALCULADORAS TRIBUTARIAS v4.5.html`**Atualizado para a legislação vigente em:** 2026

* * *

## 1. Visão geral

A ferramenta é um conjunto de calculadoras tributárias reunidas em uma única página HTML autocontida. Foi concebida para uso por profissionais da área jurídica/contábil em estimativas e conferências rápidas, com transparência total do cálculo (memória de cálculo passo a passo) e referência às normas aplicáveis.

São sete calculadoras, organizadas em abas:

1. **Lucro Presumido – PJ**
2. **Simples Nacional**
3. **Ganho de Capital (Imóveis)**
4. **IRPF Locação**
5. **IRPF Produtor Rural** (IRPF da atividade rural + Funrural + IRPF-Mínimo isolado)
6. **IRPF-Mínimo** (tributação mínima das altas rendas, considerando todos os rendimentos)
7. **Serv. Méd. X Serv. Hospitalar** (comparativo de lucro presumido: serviços hospitalares × serviços médicos)

Todos os cálculos são **estimativas** destinadas a apoio profissional e não substituem a apuração oficial nem dispensam a conferência por contador/advogado responsável.

* * *

## 2. Arquitetura técnica

* **Tecnologia:** HTML5 + CSS3 + JavaScript puro (vanilla), sem frameworks nem dependências externas.
* **Arquivo único:** todo o código (marcação, estilo e lógica) está embutido em um único `.html`. Não há requisições de rede.
* **Funcionamento 100% offline:** basta abrir o arquivo no navegador (duplo clique). Nenhum dado é enviado para servidores; tudo é processado localmente no navegador.
* **Encapsulamento:** toda a lógica roda dentro de uma IIFE (`(function(){ ... })()`), evitando poluição do escopo global.
* **Compatibilidade:** navegadores modernos (Chrome, Edge, Firefox, Safari). Usa `Intl.NumberFormat('pt-BR')` para formatação monetária.
* **Responsividade:** layout adaptado para telas estreitas (≤ 680px) — barras de entrada empilham e as tabelas viram cartões (layout "stacked"), eliminando rolagem horizontal no celular.
* **Impressão:** folha de estilo de impressão dedicada (`@media print`), com tema claro e ocultação dos botões de ação.

### 2.1 Organização do código

* **Formatadores e utilitários:** `BRL`/`DEC` (Intl), `fmt`, `pct`, `parseBR` (converte texto "1.234,56" em número), `parseMY` (interpreta "mm/aaaa").
* **Máscaras de entrada** (ver seção 4.1).
* **Por calculadora:** uma função de cálculo/renderização (`recXxx`) e uma função de memória de cálculo (`memoXxx`).
* **Inicialização:** ao carregar, restaura estado salvo (se houver), constrói as tabelas dinâmicas e recalcula tudo.

* * *

## 3. As calculadoras

> Em todas as abas, ao final, há os botões/recursos comuns: **tabela de faixas**, **métricas-resumo**, **botão "Gerar memória de cálculo"** e **notas com a base legal hiperlinkada**.

### 3.1 Lucro Presumido – PJ

**Finalidade:** estimar a carga tributária federal de empresa optante pelo lucro presumido, já com a **presunção majorada** da LC 224/2025.

**Entradas:** período (mensal/trimestral/anual) e uma ou mais linhas de receita, cada uma com a atividade (define o percentual de presunção) e o valor.

**Regras:**

* Limite da presunção original: R$ 5.000.000,00/ano (proporcional: R$ 1.250.000,00/trimestre).
* Sobre o **excedente** ao limite, a presunção é majorada em 10% (× 1,10), rateado proporcionalmente entre as receitas.
* IRPJ = 15% da base presumida; CSLL = 9% da base presumida.
* Adicional de IRPJ = 10% sobre a parcela da base de IRPJ que exceder R$ 20.000,00 × nº de meses do período.
* PIS = 0,65% e COFINS = 3% sobre a receita bruta (regime cumulativo).

**Base legal:** Lei 9.249/1995 (arts. 15 e 20), Lei 9.430/1996, LC 224/2025 e IN RFB 2.306/2026.

**Toggle "Apenas majoração LC 224".** quando ativado, o quadro passa a exibir somente IRPJ, adicional de IRPJ e CSLL, e acrescenta uma linha destacada com o **valor acrescido em razão da presunção majorada** (diferença entre o IRPJ+CSLL com a majoração de 10% sobre o excedente e o que seria devido sem a majoração). O acréscimo também é exibido como métrica dedicada, que **só aparece quando o toggle está ativado**. Com o toggle ligado, a memória de cálculo ganha uma **seção dedicada** ao cálculo da majoração (excedente ao limite, acréscimo nas bases de IRPJ e CSLL e o Δ de cada tributo, culminando no acréscimo total). Por padrão, o toggle vem desligado (cálculo completo) e a aba inicia com **uma única linha de receita**.

**Limitações:** não considera deduções da receita, retenções na fonte, ISS/ICMS nem segregações específicas.

### 3.2 Simples Nacional

**Finalidade:** calcular a alíquota efetiva e o DAS do mês.

**Entradas:** anexo/atividade (ou CNAE, que sugere o anexo), RBT12 (receita bruta dos últimos 12 meses), receita do mês e — quando sujeito ao Fator R — a folha de 12 meses.

**Regras:**

* A faixa é definida pelo **RBT12**, não pela receita do mês.
* Alíquota efetiva = (RBT12 × alíquota nominal − parcela a deduzir) ÷ RBT12.
* DAS do mês = receita do mês × alíquota efetiva.
* **Fator R:** se a folha de 12 meses ≥ 28% do RBT12, atividades sujeitas migram do Anexo V para o Anexo III.
* **Correlação CNAE → Anexo:** há uma base embutida (~130 CNAEs) que, ao informar o código, sugere o anexo; os demais podem ser selecionados manualmente.

**Base legal:** LC 123/2006 (art. 18) e Resolução CGSN nº 140/2018. Tabelas dos Anexos I a V vigentes em 2026.

### 3.3 Ganho de Capital (Imóveis)

**Finalidade:** apurar o IRPF sobre o ganho de capital na alienação de imóveis, reproduzindo a lógica do programa GCAP da Receita.

**Entradas:** por imóvel — custo de aquisição, valor de venda e mês de aquisição (mm/aaaa); e a data da venda.

**Regras (Decreto 9.580/2018 — RIR, arts. 148 a 153):**

* **Art. 148** — Ganho bruto = valor de venda − custo de aquisição.
* **Art. 149** — Redução para aquisições até 31/12/1988 (percentual fixo decrescente).
* **Art. 150** — Fatores de redução:
  * **FR1** = 1 ÷ 1,0060^m1, sendo m1 os meses de jan/1996 (ou aquisição, se posterior) até nov/2005;
  * **FR2** = 1 ÷ 1,0035^m2, sendo m2 os meses de dez/2005 (ou aquisição, se posterior) até a venda;
  * Base de cálculo = ganho (após art. 149) × FR1 × FR2.
* **Art. 153** — Alíquota: 15% para alienações até 2016; a partir de 2017, progressiva (15% até R$ 5 mi; 17,5% de 5 a 10 mi; 20% de 10 a 30 mi; 22,5% acima de 30 mi).

**Observação importante sobre a contagem de meses:** os fatores FR1/FR2 contam os meses de forma **inclusiva** (computando o próprio mês de aquisição), em conformidade com o programa GCAP da RFB. Exemplo validado: aquisição 07/2012, venda 04/2025 → 154 meses no FR2 → redução de 41,611950% → resultado idêntico ao GCAP.

### 3.4 IRPF Locação

**Finalidade:** calcular o IRPF mensal sobre aluguéis (carnê-leão).

**Entradas:** locatário (PF = carnê-leão / PJ = retido na fonte), aluguel bruto, despesas dedutíveis quando ônus do locador (IPTU, condomínio, taxa de administração) e a dedução da base (dependentes + pensão, **ou** desconto simplificado).

**Regras:**

* Aluguel líquido (rendimento tributável) = aluguel bruto − IPTU − condomínio − administração.
* Base de cálculo = rendimento tributável − deduções.
* Imposto pela **tabela progressiva mensal** (Lei 15.191/2025).
* **Redução mensal** (Lei 15.270/2025): rendimento até R$ 5.000 zera o imposto (redução de até R$ 312,89); de R$ 5.000,01 a R$ 7.350,00 a redução é R$ 978,62 − 0,133145 × rendimento, decrescente até zerar.

**Base legal:** RIR/2018 (arts. 41 e 689); Lei 15.191/2025 (tabela mensal) e Lei 15.270/2025 (redução).

### 3.5 IRPF Produtor Rural

Reúne, na mesma aba, três blocos para o produtor rural pessoa física.

**A) Resultado da atividade rural e IRPF** (Decreto 9.580/2018, arts. 50 a 64):

* Formas de apuração: **Resultado presumido** (20% da receita — art. 63), **Livro-caixa** (receita − despesas de custeio e investimentos — art. 56, com compensação de prejuízos de anos anteriores — art. 58) ou **Arbitramento** (20% por falta de livro-caixa — art. 53, §2º).
* IRPF: residente no Brasil → tabela anual 2026 com desconto simplificado e redução da Lei 15.270/2025; residente no exterior → 15% fixo (art. 64).
* Alíquota efetiva exibida sobre a **receita bruta anual**.

**B) Contribuição previdenciária (Funrural)** — incide sobre a receita da comercialização da produção (IN RFB 2.110/2022; Lei 8.212/1991, art. 25):

* **Produtor rural PF (contribuinte individual):** 1,50% até mar/2026; **1,63%** a partir de abr/2026 (majoração da LC 224/2025) — composição: previdenciária + GILRAT/RAT + SENAR.
* **Segurado especial:** permanece em **1,50%** (não afetado pela majoração).

**C) IRPF-Mínimo isolado:** calcula o IRPFM considerando **apenas o resultado da atividade rural** (sem demais rendimentos), para estimar a parcela atribuível à produção rural. Em regra, o IRPF regular do resultado rural já supera o mínimo, resultando em complemento zero — o IRPFM tende a incidir apenas quando combinado a rendimentos de baixa tributação (tratado na aba específica).

**D) Tributação consolidada:** quadro com IRPF + Funrural + IRPF-Mínimo, valores e alíquotas efetivas, e a carga total no ano.

### 3.6 IRPF-Mínimo (IRPFM)

**Finalidade:** estimar a tributação mínima das altas rendas instituída pela Lei 15.270/2025, considerando **todos os rendimentos** do contribuinte.

**Entradas:** rendimentos tributáveis (exceto rural); atividade rural (com seletor entre "receita bruta" — que aplica o presumido de 20% — ou "resultado tributável", usado diretamente); lucros e dividendos; demais rendimentos incluídos; IRRF retido; redutor de dividendos (opcional).

**Regras:**

* **Base** = soma do resultado da atividade rural e dos demais rendimentos do ano (tributáveis, isentos e de tributação exclusiva), com exclusões legais (ganhos de capital fora de bolsa, poupança, títulos imobiliários e do agronegócio, lucros apurados até 31/12/2025 etc.).
* **Alíquota mínima:** 0% até R$ 600.000; entre R$ 600.000,01 e R$ 1.200.000 cresce linearmente pela fórmula (rendimento ÷ 60.000) − 10, de 0% a 10%; acima de R$ 1.200.000, 10% fixo.
* **IRPFM apurado** = base × alíquota mínima.
* **IRPF regular estimado:** quadro próprio calcula, sobre os rendimentos tributáveis + resultado rural, o imposto que já seria pago na declaração (tabela anual + desconto simplificado + redução), que é deduzido do IRPFM.
* **Complemento a pagar** = máx(IRPFM apurado − IRPF regular estimado − IRRF − redutor; 0).

**Base legal:** Lei 15.270/2025 (conversão do PL 1.087/2025). **Norma ainda pendente de regulamentação** — o redutor de dividendos é informado manualmente.

### 3.7 Serv. Méd. X Serv. Hospitalar (comparativo)

**Finalidade:** comparar, no lucro presumido, a tributação da **mesma receita** classificada como serviços hospitalares e como serviços profissionais (médicos), evidenciando a diferença de carga.

**Entradas:** período (mensal/trimestral/anual) e a receita bruta (valor único).

**Regras:** aplica o mesmo cálculo do lucro presumido (seção 3.1) em duas hipóteses simultâneas:

* **Serviços hospitalares:** presunção IRPJ 8% / CSLL 12%;
* **Serviços profissionais (médicos):** presunção IRPJ 32% / CSLL 32%.

Para cada hipótese apura IRPJ (15%), adicional (10% sobre a base que exceder R$ 20.000 × nº de meses), CSLL (9%), PIS (0,65%) e COFINS (3%), com a majoração de 10% da presunção sobre o excedente ao limite (LC 224/2025). Um terceiro quadro **comparativo** apresenta, por tributo e no total, os valores das duas hipóteses e a **diferença** (médico − hospitalar).

**ISS.** cada quadro inclui uma linha de ISS: nos **serviços hospitalares**, incide sobre a receita bruta com alíquota municipal selecionável de 2% a 5%; nos **serviços profissionais** (sociedade uniprofissional), é um valor fixo por profissional — informam-se o nº de profissionais e o valor fixo por profissional (conforme o período), sendo o ISS o produto dos dois.

**Base legal:** art. 15, caput e §1º, III, "a", e art. 20 da Lei 9.249/1995 (redação da Lei 11.727/2008); Lei 9.430/1996; LC 224/2025 e IN RFB 2.306/2026; IN RFB nº 1.700/2017 (art. 33, §1º, II, "a", e §3º); Soluções de Consulta COSIT nº 36/2016 e nº 322/2017; RDC Anvisa nº 50/2002 (atribuições 1 a 4).

**Observação (requisitos da presunção reduzida 8%/12%).** o prestador deve estar constituído como **sociedade empresária** (a EIRELI foi afastada pela SC COSIT nº 322/2017) e **atender às normas da Anvisa** — ambientes conforme as atribuições 1 a 4 da RDC nº 50/2002, comprovados por alvará da vigilância sanitária. As simples consultas médicas em consultório ficam sujeitas à presunção de 32%.

**Comparação visual.** o total do regime mais favorável (menor carga) é destacado em verde e o outro em vermelho, nos cartões-resumo e no quadro comparativo. Um gráfico de barras (SVG) exibe os dois totais lado a lado, com legenda indicando o regime mais vantajoso e a economia no período.

* * *

## 4. Recursos transversais

### 4.1 Máscaras de entrada em tempo real

* **Monetária:** os campos de valor formatam-se enquanto se digita, acumulando centavos da direita para a esquerda (ex.: digitar `600000000` exibe `6.000.000,00`). Implementada com um listener em **fase de captura**, que formata o valor **antes** do recálculo, garantindo consistência inclusive nos campos criados dinamicamente (linhas de imóveis e de receitas). O campo de dependentes é exceção (número inteiro).
* **Data (mm/aaaa):** a barra "/" é inserida automaticamente após os dois primeiros dígitos.

### 4.2 Memórias de cálculo

Cada aba gera uma memória de cálculo auditável, em formato padronizado:

* **Dados informados** — relação dos valores e parâmetros digitados;
* **Etapas do cálculo** — cada passo com rótulo, fórmula (com os números substituídos) e resultado destacado;
* **Resultado final** — total/tributo apurado e alíquota efetiva.

A memória integra-se à impressão da página.

### 4.3 Hiperlinks da legislação

As citações de leis e atos normativos são hiperlinks para os repositórios oficiais (Planalto e o portal de normas da Receita Federal), abrindo em nova aba.

### 4.4 Salvar e restaurar (continuar de onde parou)

* O botão **Salvar** (ícone de disquete) baixa um arquivo HTML idêntico, porém com **todo o estado preenchido embutido** (campos, tabelas de imóveis/receitas, aba ativa e título do caso).
* Ao reabrir esse arquivo, a calculadora **restaura automaticamente** todos os dados.
* O estado é serializado em JSON e gravado em um bloco `<script id="estadoSalvo">` no `<head>`, lido na inicialização.
* O nome do arquivo baixado incorpora o título do caso (ex.: "Calculadoras Tributarias - Cliente X.html").

### 4.5 Identificação do caso

Caixa de texto ao lado do título, para identificar a que caso se referem os cálculos. É salva junto com o estado e nomeia o arquivo ao salvar.

### 4.6 Outros

* **Impressão:** botão "Imprimir" gera versão em tema claro, ocultando os botões de ação (Salvar/Imprimir) e a barra de abas; o título do caso é impresso como texto.
* **Rodapé:** exibe a versão da calculadora em todas as abas.

* * *

## 5. Tabelas de referência embutidas (2026)

### 5.1 IRPF — tabela mensal (Lei 15.191/2025)

| Base de cálculo mensal | Alíquota | Dedução |
| --- | --- | --- |
| Até R$ 2.428,80 | isento | —   |
| 2.428,81 a 2.826,65 | 7,5% | R$ 182,16 |
| 2.826,66 a 3.751,05 | 15,0% | R$ 394,16 |
| 3.751,06 a 4.664,68 | 22,5% | R$ 675,49 |
| Acima de 4.664,68 | 27,5% | R$ 908,73 |

Dependente: R$ 189,59/mês · Desconto simplificado: R$ 607,20/mês.

### 5.2 IRPF — tabela anual (Lei 15.191/2025)

| Base de cálculo anual | Alíquota | Dedução |
| --- | --- | --- |
| Até R$ 29.145,60 | isento | —   |
| 29.145,61 a 33.919,80 | 7,5% | R$ 2.185,92 |
| 33.919,81 a 45.012,60 | 15,0% | R$ 4.729,91 |
| 45.012,61 a 55.976,16 | 22,5% | R$ 8.105,85 |
| Acima de 55.976,16 | 27,5% | R$ 10.904,66 |

Desconto simplificado anual: R$ 17.640,00.

### 5.3 Funrural — produtor rural PF (sobre a comercialização)

| Componente | Até mar/2026 | A partir de abr/2026 |
| --- | --- | --- |
| Previdenciária | 1,20% | 1,32% |
| GILRAT/RAT | 0,10% | 0,11% |
| SENAR | 0,20% | 0,20% |
| **Total** | **1,50%** | **1,63%** |

Segurado especial: mantém 1,50%.

* * *

## 6. Limitações e avisos gerais

* Todas as calculadoras produzem **estimativas**; não substituem a apuração oficial (GCAP, PGDAS-D, DIRPF, eSocial etc.) nem a análise por profissional habilitado.
* Os cálculos do IRPF-Mínimo e do Funrural majorado baseiam-se em normas recentes, parte delas **pendente de regulamentação** — sujeitas a ajuste.
* O IRPF-Mínimo depende do conjunto dos rendimentos do contribuinte; a aba específica exige a informação do rendimento global e do IR regular para apuração do complemento.
* A correlação CNAE → Anexo do Simples cobre uma amostra significativa, porém não exaustiva; havendo dúvida, selecione o anexo manualmente.

* * *

## 7. Histórico de versões

**v4.5**

* Aba "Lucro Presumido – PJ": a métrica de acréscimo pela majoração LC 224/2025 passa a aparecer apenas com o toggle ativado; com o toggle ligado, a memória de cálculo inclui uma seção dedicada ao cálculo da majoração (excedente, acréscimo de base de IRPJ/CSLL e Δ por tributo).

**v4.4**

* Aba "Lucro Presumido – PJ": passa a iniciar com uma única linha de receita; novo toggle "Apenas majoração LC 224" que restringe o quadro a IRPJ/adicional/CSLL e destaca o acréscimo tributário causado pela presunção majorada da LC 224/2025 (linha no quadro e métrica dedicada). O estado do toggle é salvo/restaurado junto com o caso.

**v4.3**

* Impressão: correção dos botões Salvar/Imprimir que apareciam pretos (agora ocultados na impressão); a barra de abas deixa de ser impressa; o campo de identificação do caso é impresso como texto limpo.

**v4.2**

* Aba "Serv. Méd. X Serv. Hospitalar": realce por cores do regime mais favorável (verde) e do menos favorável (vermelho) nos totais; gráfico de barras (SVG) comparando os dois regimes, com legenda da economia no período.

**v4.1**

* Aba "Serv. Méd. X Serv. Hospitalar": inclusão do ISS nos dois quadros (hospitalar sobre a receita, alíquota selecionável de 2% a 5%; profissionais com valor fixo por profissional × nº de profissionais) e no comparativo.
* Notas de fundamentação legal ampliadas com os dispositivos e soluções de consulta do parecer (art. 15, §1º, III, "a", e art. 20 da Lei 9.249/1995; Lei 11.727/2008; IN RFB 1.700/2017, art. 33; SC COSIT nº 36/2016 e nº 322/2017; RDC Anvisa nº 50/2002).
* Correção da posição do rodapé de versão (agora ao fim da página em todas as abas).

**v4.0**

* Nova aba "Serv. Méd. X Serv. Hospitalar": cálculo simultâneo do lucro presumido como serviços hospitalares (8%/12%) e como serviços médicos (32%/32%), com quadro comparativo, memória de cálculo e fundamentação legal.
* Correção do recurso Salvar: a marca de estado passou a ser fragmentada no código, evitando que a limpeza do estado antigo cortasse o próprio script ao gerar o arquivo salvo.

**v3.0**

* Seis calculadoras (Lucro Presumido, Simples Nacional, Ganho de Capital, IRPF Locação, IRPF Produtor Rural, IRPF-Mínimo).
* Correção da contagem inclusiva de meses no FR1/FR2 (alinhamento ao GCAP).
* IRPF-Mínimo (isolado na aba rural e completo em aba própria), com quadro de IRPF regular estimado.
* Funrural com majoração da LC 224/2025 (abr/2026).
* Memórias de cálculo padronizadas; hiperlinks legais sempre em azul.
* Máscaras monetária e de data em tempo real.
* Salvar/restaurar estado em HTML; caixa de identificação do caso; rodapé com a versão.
