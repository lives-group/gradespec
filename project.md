# Título

GradeSpec: Uma Linguagem para Especificação de Critérios de Correção em Juízes
Online

# Resumo

A avaliação automática de exercícios de programação é realizada por sistemas
denominados juízes online, que comparam a saída produzida pelo programa do aluno
com uma saída esperada previamente definida. A maioria desses sistemas baseia-se
em comparação textual exata, desconsiderando variações semânticas relevantes,
como diferenças de arredondamento em valores numéricos e variações legítimas em
partes dinâmicas da resposta. Trabalhos recentes propõem processos de correção
mais sofisticados, que distinguem componentes estáticos de dinâmicos e tratam
valores numéricos com semântica própria, mas o fazem por meio de estruturas
tabulares definidas manualmente ou de forma semiautomatizada, sem uma linguagem
apropriada que permita especificar, verificar e reutilizar critérios de
correção.

Este projeto propõe a concepção, definição formal e implementação de GradeSpec,
uma linguagem declarativa para especificação de critérios de correção de
exercícios de programação. A linguagem permite ao professor descrever, em um
único artefato textual, o padrão esperado para cada linha da saída do programa e
as ações semânticas de pontuação associadas a cada fragmento desse padrão. O
fundamento formal de GradeSpec é o formalismo das Parsing Expression Grammars
(PEGs), que modela o problema de correção como casamento de padrão
determinístico com ações semânticas. A escolha de PEGs é motivada pela natureza
composicional do problema: a saída esperada de um programa é uma sequência
estruturada de fragmentos de tipos distintos (texto fixo, palavras-chave
variáveis, valores numéricos), e PEGs oferecem operadores sequência, escolha
ordenada e predicados lookahead que expressam diretamente essa estrutura, sem as
ambiguidades inerentes às gramáticas livres de contexto.

O projeto prevê a formalização da sintaxe e da semântica de GradeSpec, o
desenvolvimento de um software de avaliação que interpreta especificações
escritas na linguagem e avalia submissões de alunos de forma consistente, e a
validação experimental da abordagem em um contexto real de disciplina
introdutória de programação. Os resultados esperados incluem a especificação
formal da linguagem, uma implementação de referência de código aberto e
evidências empíricas do impacto da abordagem na qualidade do feedback
pedagógico.

# Palavras-chave

juízes online, parsing expression grammars, correção automática, avaliação de
programação, linguagens de especificação, análise dinâmica

# Introdução

As disciplinas introdutórias de programação ocupam posição central nos
currículos de cursos de computação e áreas afins. Seu objetivo é desenvolver o
raciocínio lógico e a capacidade de abstração necessários à formação do
profissional de tecnologia. Contudo, o alto índice de reprovação e evasão nessas
disciplinas é um fenômeno bem documentado [Mirzayanov et al., 2020],
frequentemente associado à dificuldade dos alunos em obter feedback rápido e
preciso sobre suas soluções.

Como resposta a essa necessidade, foram desenvolvidos os juízes online, sistemas
que permitem ao aluno submeter seu programa e obter uma avaliação automática em
tempo hábil [Bez et al., 2014; CodeBench, 2026; Mirzayanov et al., 2020]. O
mecanismo central de avaliação adotado pela maioria desses sistemas é a análise
dinâmica [Ball, 1999]: o programa é executado sobre entradas predefinidas e a
saída produzida é comparada com uma saída esperada. Essa comparação é, em geral,
realizada por casamento exato de strings ou por medidas de similaridade textual
[Lisbach e Meyer, 2013].

A abordagem por casamento exato, embora eficaz no contexto da programação
competitiva (para o qual a maioria dos juízes online foi originalmente concebida
[Messer et al., 2024]), apresenta limitações pedagógicas relevantes. Pequenas
variações que não comprometem a correção lógica da solução, como diferenças de
acentuação, espaçamento ou arredondamento de valores numéricos, resultam em
penalizações que frustram e desmotivam alunos em fase inicial de aprendizagem.
Por outro lado, a similaridade textual irrestrita pode atribuir notas elevadas a
respostas numericamente incorretas que se assemelham textualmente à resposta
correta: por exemplo, a string "5234" possui alta similaridade com "1234",
embora os valores divirjam em quatro mil unidades.

O trabalho de referência deste projeto [opCoders, 2026] propõe um processo de
correção pedagogicamente mais coerente, que diferencia fragmentos estáticos de
dinâmicos na saída esperada, trata valores numéricos com semântica de
arredondamento e penalidades proporcionais, e gera marcadores que evidenciam ao
aluno a natureza de cada divergência encontrada. Esse trabalho demonstra
experimentalmente que tal abordagem produz avaliações mais justas e feedback
mais útil. Contudo, a definição dos critérios de correção ainda é feita de forma
tabular e semiautomatizada, sem uma linguagem formal que permita ao professor
especificá-los diretamente, verificá-los antes da aplicação e reutilizá-los
sistematicamente.

A ausência de uma linguagem formal para critérios de correção cria uma lacuna
entre a teoria da avaliação semântica e sua aplicação prática: sem sintaxe e
semântica precisamente definidas, não é possível garantir que critérios
distintos sejam interpretados de forma consistente, detectar erros de
especificação antes da avaliação, ou integrar automaticamente os critérios a
ferramentas de geração assistida por inteligência artificial.

O problema de descrever a estrutura esperada da saída de um programa e extrair
dela fragmentos para avaliação independente é, em sua essência, um problema de
casamento de padrão estruturado, isto é, um problema de parsing. As Parsing
Expression Grammars (PEGs) [Ford, 2004] são um formalismo de reconhecimento de
linguagens que se distingue das gramáticas livres de contexto (GLCs) por sua
semântica determinística: o operador de escolha `/` é ordenado, e a primeira
alternativa que casa é aceita, sem backtracking não determinístico. Essa
propriedade elimina ambiguidades e torna o comportamento do parser totalmente
previsível. Além disso, PEGs incorporam predicados lookahead (`&e` e `!e`) que
permitem expressar restrições contextuais sem consumir entrada, e seu formalismo
é fechado sob composição sequencial — exatamente a estrutura de uma linha de
saída composta por fragmentos de tipos distintos.

A abordagem de atribuir ações semânticas a nós de gramáticas para o cálculo de
valores durante o parse é consolidada na teoria de compiladores [Aho et al.,
2006], sob o nome de gramáticas com atributos. A combinação de PEGs com ações
semânticas de pontuação fornece, portanto, a base teórica adequada para uma
linguagem de especificação de critérios de correção.

Este projeto de pesquisa propõe a concepção e o desenvolvimento de GradeSpec,
uma linguagem declarativa fundamentada em PEGs para especificação de critérios
de correção de exercícios de programação. A investigação abrange a definição
formal da sintaxe e da semântica da linguagem, o desenvolvimento de um motor de
avaliação que a interprete, e a validação experimental em contexto real de
ensino.

# Objetivos

O objetivo geral deste projeto é conceber, formalizar e implementar GradeSpec,
uma linguagem de especificação de critérios de correção para juízes online,
fundamentada em Parsing Expression Grammars e dotada de ações semânticas de
pontuação para avaliação de saídas de programas de alunos.

Os objetivos específicos são:

**1. Revisar e sistematizar o estado da arte em avaliação automática de
exercícios de programação** Realizar levantamento bibliográfico abrangente sobre
juízes online com foco pedagógico, técnicas de análise dinâmica de programas,
métricas de similaridade textual e numérica aplicadas à correção automática, e
abordagens de feedback adaptativo. Identificar as principais limitações dos
sistemas atuais quanto à expressividade dos critérios de correção e à
consistência das avaliações produzidas.

**2. Dominar os fundamentos teóricos de Parsing Expression Grammars** Estudar em
profundidade o formalismo PEG [Ford, 2004], incluindo sua semântica formal de
reconhecimento, as propriedades do operador de escolha ordenada, os predicados
lookahead e a relação com autômatos de pilha determinísticos. Estudar gramáticas
com atributos [Aho et al., 2006] como modelo de ações semânticas associadas a
regras gramaticais, e investigar sua combinação com PEGs em ferramentas como
PEG.js, Lark e ANTLR.

**3. Definir formalmente a sintaxe de GradeSpec** Especificar a gramática
completa de GradeSpec como uma PEG, cobrindo os construtores para fragmentos
estáticos, dinâmicos, numéricos, de entrada e de linha em branco; as anotações
de peso e penalidade; a estrutura de blocos de casos de teste; e os mecanismos
de composição para saídas multilinhas e padrões repetitivos. Garantir que a
gramática seja não ambígua e analisável em tempo linear, propriedades
decorrentes do formalismo PEG.

**4. Definir a semântica formal de GradeSpec** Especificar, em notação
matemática precisa, as ações semânticas associadas a cada tipo de fragmento: a
função de pontuação de fragmentos estáticos baseada em distância de Levenshtein
normalizada; a função de pontuação de fragmentos dinâmicos baseada em
containment check com fallback para Levenshtein com penalidade; e a função de
pontuação numérica baseada em igualdade com arredondamento semântico. Definir
formalmente as noções de diferença penalizante e não penalizante e o mecanismo
de geração de marcadores de feedback.

**5. Implementar um motor de avaliação para GradeSpec** Desenvolver uma
implementação de referência em Haskell que: (i) realize o parse de arquivos
`.gspec` segundo a gramática definida; (ii) aplique as ações semânticas sobre a
saída real do aluno, produzindo nota por critério e nota total; e (iii) gere a
saída anotada com marcadores de diferença que permitam ao frontend do juiz
online evidenciar ao aluno a natureza e o impacto de cada divergência
encontrada.

**6. Integrar geração automática de especificações via modelos de linguagem**
Investigar o uso de Large Language Models (LLMs) para a geração automática de
arquivos `.gspec` a partir de enunciados de exercícios e casos de teste de
exemplo, reduzindo o esforço manual do professor. Avaliar a qualidade e a
completude das especificações geradas e definir diretrizes de prompt engineering
para esse fim.

**7. Validar a abordagem experimentalmente** Reproduzir, usando o motor
GradeSpec como backend de avaliação, os experimentos do trabalho de referência
[opCoders, 2026] sobre avaliações reais de uma disciplina introdutória de
programação, verificando a equivalência dos resultados e quantificando os
benefícios da especificação formal quanto à consistência, detectabilidade de
erros de critério e facilidade de manutenção.

**8. Produzir e disseminar resultados científicos** Documentar os resultados em
relatório técnico, submeter artigo científico a evento ou periódico da área
(e.g., SBIE, WEI, SIGCSE, COMPSAC), e disponibilizar publicamente a
especificação formal e a implementação de referência em repositório aberto.

# Justificativa/Relevância

A qualidade do feedback fornecido por juízes online tem impacto direto na
aprendizagem de programação. Sistemas que penalizam igualmente um erro de
acentuação e um erro de lógica comunicam ao aluno uma visão distorcida da
correção de seu programa, dificultando a identificação e a superação das
dificuldades reais. A pesquisa em avaliação automática com foco pedagógico é,
portanto, uma necessidade premente para o ensino de computação no Brasil e no
mundo [Kurniawan et al., 2023; Messer et al., 2024].

Trabalhos recentes demonstraram experimentalmente que processos de correção
semântica, que diferenciam fragmentos de tipos distintos e tratam números com
ações específicas, produzem avaliações significativamente mais justas: em um
estudo com 6.314 submissões reais, 43,46% das notas foram afetadas pela mudança
de abordagem [opCoders, 2026]. Esses resultados evidenciam tanto o impacto
prático da questão quanto a necessidade de consolidar essa abordagem sobre bases
formais sólidas.

A formalização por meio de uma linguagem com sintaxe e semântica precisas
oferece vantagens que vão além da clareza conceitual. Do ponto de vista da
engenharia de software educacional, uma linguagem formal permite: a detecção de
erros de especificação antes da avaliação (por meio do parse do próprio arquivo
`.gspec`), garantindo que critérios mal formados não produzam avaliações
incorretas silenciosamente; a reutilização e o compartilhamento de critérios
entre professores e instituições; e a integração direta com ferramentas de
geração assistida por IA, que podem produzir e validar arquivos `.gspec` a
partir de enunciados, sem intervenção manual. Do ponto de vista científico, a
formalização via PEGs conecta o problema de correção automática a um corpo
teórico consolidado — a teoria de parsing e linguagens formais —, abrindo
caminho para resultados de correção, completude e complexidade computacional
sobre o processo de avaliação.

A escolha de PEGs como fundamento não é apenas pragmática. PEGs são o formalismo
adequado para o domínio porque o problema de reconhecer a estrutura de uma linha
de saída e extrair seus fragmentos é precisamente um problema de parsing
determinístico: a saída do aluno ou casa com o padrão esperado (possivelmente de
forma aproximada) ou não casa, e a ordem de tentativa das alternativas é
semanticamente significativa, exatamente o que o operador de escolha ordenada de
PEGs modela. A integração de ações semânticas de pontuação a esse formalismo é
natural e bem fundamentada na teoria de gramáticas com atributos.

Por fim, o desenvolvimento desta pesquisa contribui para a formação de recursos
humanos em linguagens formais e teoria de compiladores aplicados ao domínio de
tecnologia educacional, uma combinação de áreas com crescente relevância prática
e ainda escassamente explorada no Brasil.

# Atividades/Metodologias

A pesquisa será conduzida em etapas progressivas, combinando revisão
bibliográfica, desenvolvimento teórico-formal e implementação experimental.

**Etapa 1: Revisão bibliográfica e nivelamento teórico (meses 1–3)**

Estudo dos fundamentos necessários ao projeto:

- Juízes online e avaliação automática de programação: Beecrowd [da Cruz et al.,
  2022], CodeBench [CodeBench, 2026], URI Online Judge [Bez et al., 2014;
  Selivon et al., 2015], Codeforces [Mirzayanov et al., 2020], e o sistema com
  foco pedagógico que serve de referência ao projeto [opCoders, 2026];
- Análise dinâmica de programas [Ball, 1999] e métricas de similaridade textual,
  em especial a distância de Levenshtein [Lisbach e Meyer, 2013];
- Parsing Expression Grammars: formalismo [Ford, 2004], relação com GLCs e
  autômatos de pilha, propriedades de determinismo e complexidade [Ford, 2002];
- Gramáticas com atributos e ações semânticas em compiladores [Aho et al.,
  2006];
- Ferramentas de implementação de parsers PEG utilizando Megaparsec (Haskell);
- Uso de LLMs para geração de especificações formais: técnicas de prompt
  engineering e avaliação de qualidade de saídas estruturadas.

Serão lidos e fichados todos os trabalhos de referência do projeto. Um protótipo
exploratório de sintaxe será esboçado ao final desta etapa para guiar as
discussões das etapas seguintes.

**Etapa 2: Definição formal da sintaxe de GradeSpec (meses 2–5)**

Com base nos fundamentos estudados na Etapa 1:

- Identificação e classificação dos tipos de fragmento necessários para cobrir
  os padrões de saída dos exercícios introdutórios de programação: texto
  estático, palavra ou frase dinâmica com alternativas, valor numérico inteiro
  ou real com tolerância de arredondamento, prompt de entrada e linha em branco;
- Definição da gramática completa de GradeSpec como PEG, cobrindo a estrutura de
  arquivos (cabeçalho, blocos de casos de teste, seções de entrada e saída), a
  sintaxe dos construtores de fragmento, as anotações de peso e penalidade e os
  comentários;
- Extensão da gramática para suporte a padrões multilinhas e blocos de
  repetição, necessários para exercícios com saída iterativa (e.g., tabelas,
  sequências);
- Verificação das propriedades da gramática: ausência de recursão à esquerda,
  determinismo do operador de escolha, linearidade do tempo de parse;
- Produção de um documento de especificação formal da sintaxe, incluindo a PEG
  completa comentada e um conjunto de exemplos de arquivos `.gspec` válidos e
  inválidos para fins de teste.

**Etapa 3: Definição formal da semântica de GradeSpec (meses 4–7)**

Com base na sintaxe definida na Etapa 2 e nos resultados do trabalho de
referência [opCoders, 2026]:

- Especificação da semântica de reconhecimento: para cada tipo de fragmento,
  definição da sub-PEG que reconhece na saída do aluno a porção correspondente,
  incluindo o token numérico baseado na expressão regular `(-?\d+\.\d+|-?\d+)` e
  o mecanismo de extração de fragmentos textuais por delimitação;
- Especificação das ações semânticas de pontuação: formalização matemática das
  funções `score_static`, `score_dynamic` e `score_number`, incluindo a
  definição precisa de diferença penalizante versus não penalizante (acentuação,
  pontuação, espaçamento) e a função de arredondamento semântico
  `round_to_min_decimals`;
- Especificação do mecanismo de geração de marcadores de diferença: definição
  formal dos marcadores `@<<@`, `@>>@`, `@^^@`, `@===@` e `@---@` e das
  condições de sua inserção em função do tipo e do impacto de cada divergência;
- Definição da semântica composicional da nota total: acumulação por critério,
  ponderação por tamanho de fragmentos textuais e normalização para o intervalo
  [0, 10];
- Produção de um documento de especificação formal da semântica, com a definição
  de todas as funções semânticas, seus domínios e contradomínios, e um conjunto
  de exemplos de avaliação passo a passo para os cenários dos experimentos do
  trabalho de referência.

**Etapa 4: Implementação do motor de avaliação (meses 5–8)**

Com base nas especificações das Etapas 2 e 3:

- Implementação do parser de GradeSpec em Haskell, utilizando a biblioteca
  Megaparsec com combinadores de parser, produzindo uma árvore de critérios
  estruturada;
- Implementação das funções semânticas de pontuação: distância de Levenshtein
  normalizada para fragmentos estáticos e dinâmicos; arredondamento semântico e
  comparação numérica para fragmentos numéricos; normalização de acentuação e
  pontuação para classificação de diferenças não penalizantes;
- Implementação do módulo de geração de marcadores de diferença, compatível com
  a interface de feedback do juiz online de referência;
- Desenvolvimento de uma suíte de testes unitários e de integração, cobrindo
  todos os tipos de fragmento, casos de borda (saídas vazias, linhas em branco
  inesperadas, números com muitas casas decimais) e os cenários experimentais do
  trabalho de referência;
- Disponibilização do código em repositório público com documentação de uso
  (README, exemplos de `.gspec` e instruções de integração).

**Etapa 5: Integração com geração automática via LLM (meses 7–9)**

- Desenvolvimento de prompts estruturados para geração de arquivos `.gspec` a
  partir de enunciados de exercícios e casos de teste de exemplo, utilizando
  modelos de linguagem de grande porte;
- Avaliação da qualidade das especificações geradas: parse com o motor
  implementado na Etapa 4 para detecção de erros sintáticos; comparação com
  especificações elaboradas manualmente por especialistas quanto à cobertura de
  casos de borda e à distribuição de pesos;
- Definição de diretrizes de prompt engineering para geração confiável de
  `.gspec`, incluindo exemplos few-shot e instruções de formato;
- Análise do esforço de correção manual residual necessário para as
  especificações geradas automaticamente, como indicador de maturidade da
  abordagem.

**Etapa 6: Validação experimental (meses 8–11)**

- Reprodução dos experimentos do trabalho de referência [opCoders, 2026]
  utilizando o motor GradeSpec como backend: reprocessamento das 6.314
  submissões reais com especificações `.gspec` equivalentes às tabelas
  originais, verificando equivalência numérica dos resultados;
- Experimento adicional de usabilidade com professores: avaliação da facilidade
  de escrita e leitura de especificações `.gspec` em comparação com a definição
  tabular original, por meio de questionário estruturado e análise de tempo de
  especificação;
- Análise do impacto da detecção estática de erros de critério: levantamento de
  casos em que o parse do `.gspec` teria detectado inconsistências que, no
  processo original, só se manifestariam durante a avaliação;
- Análise comparativa do desempenho computacional do motor GradeSpec em relação
  à implementação de referência, considerando o overhead do parser PEG.

**Etapa 7: Escrita e disseminação de resultados (meses 10–12)**

- Elaboração de relatório técnico detalhado contendo a especificação formal
  completa de GradeSpec (sintaxe e semântica) e a documentação da implementação;
- Escrita e submissão de artigo científico descrevendo a linguagem, sua
  fundamentação em PEGs e os resultados experimentais, direcionado a evento ou
  periódico da área (e.g., SBIE, WEI, SIGCSE, COMPSAC);
- Disponibilização pública da especificação formal, da implementação de
  referência e do conjunto de dados experimentais em repositório aberto (GitHub
  e Zenodo);
- Apresentação dos resultados em seminários e eventos acadêmicos nacionais.

# Referências Bibliográficas

[1] FORD, B. Parsing Expression Grammars: A Recognition-Based Syntactic
Foundation. In: _31st ACM SIGPLAN-SIGACT Symposium on Principles of Programming
Languages (POPL '04)_. ACM, 2004. p. 111–122.
https://doi.org/10.1145/964001.964011

[2] FORD, B. Packrat Parsing: Simple, Powerful, Lazy, Linear Time, Functional
Pearl. In: _7th ACM SIGPLAN International Conference on Functional Programming
(ICFP '02)_. ACM, 2002. p. 36–47. https://doi.org/10.1145/581478.581483

[3] BALL, T. The Concept of Dynamic Analysis. In: NIERSTRASZ, O.; LEMOINE, M.
(eds.). _Software Engineering — ESEC/FSE '99_. Lecture Notes in Computer
Science, v. 1687. Springer, 1999. p. 216–234.
https://doi.org/10.1007/3-540-48166-4_14

[4] LISBACH, B.; MEYER, V. _Linguistic Identity Matching_. Wiesbaden: Springer
Vieweg, 2013. ISBN 978-3-8348-1370-1.

[5] AHO, A. V.; LAM, M. S.; SETHI, R.; ULLMAN, J. D. _Compilers: Principles,
Techniques, and Tools_. 2. ed. Boston: Addison-Wesley, 2006. ISBN
978-0-321-48681-3.

[6] MESSER, M.; BROWN, N. C. C.; KÖLLING, M.; SHI, M. Automated Grading and
Feedback Tools for Programming Education: A Systematic Review. _ACM Transactions
on Computing Education_, v. 24, n. 1, p. 1–43, 2024.
https://doi.org/10.1145/3636515

[7] KURNIAWAN, O.; POSKITT, C. M.; AL HOQUE, I.; LEE, N. T. S.; JÉGOUREL, C.;
SOCKALINGAM, N. How Helpful Do Novice Programmers Find the Feedback of an
Automated Repair Tool? In: _2023 IEEE International Conference on Teaching,
Assessment and Learning for Engineering (TALE)_. IEEE, 2023. p. 1–6.
https://doi.org/10.1109/TALE56641.2023.10398393

[8] BEZ, J. L.; TONIN, N. A.; RODEGHERI, P. R. URI Online Judge Academic: A Tool
for Algorithms and Programming Classes. In: _9th International Conference on
Computer Science & Education (ICCSE 2014)_. IEEE, 2014. p. 149–152.
https://doi.org/10.1109/ICCSE.2014.6926445

[9] SELIVON, M.; BEZ, J. L.; TONIN, N. URI Online Judge Academic: Integração e
Consolidação da Ferramenta no Processo de Ensino/Aprendizagem. In: _Workshop
sobre Educação em Computação (WEI 2015)_, 23. Recife. Anais [...]. Porto Alegre:
SBC, 2015. p. 188–195. https://doi.org/10.5753/wei.2015.10235

[10] MIRZAYANOV, M.; PAVLOVA, O.; MAVRIN, P.; MELNIKOV, R.; PLOTNIKOV, A.;
PARFENOV, V.; STANKEVICH, A. Codeforces as an Educational Platform for Learning
Programming in Digitalization. _Olympiads in Informatics_, v. 14, p. 133–142,
2020. https://doi.org/10.15388/ioi.2020.10

[11] INSTITUTO DE COMPUTAÇÃO — UNIVERSIDADE FEDERAL DO AMAZONAS. CodeBench:
Sistema de Juiz Online para Ensino de Programação. Disponível em:
https://codebench.icomp.ufam.edu.br. Acesso em: jun. 2026.

[12] DA CRUZ, A. K. B. S.; SOARES NETO, C. S.; DA CRUZ, P. T. M. B.; TEIXEIRA,
M. A. M. Utilização da Plataforma Beecrowd de Maratona de Programação como
Estratégia para o Ensino de Algoritmos. In: _Anais Estendidos do XXI Simpósio
Brasileiro de Jogos e Entretenimento Digital (SBGames 2022)_. Porto Alegre: SBC,
2022. p. 754–764. https://doi.org/10.5753/sbgames_estendido.2022.225898

[13] opCoders. Incorporando Tolerância Numérica e Padrões Textuais Dinâmicos na
Correção Automática de Exercícios de Programação. In: _Simpósio Brasileiro de
Informática na Educação (SBIE 2026)_. SBC, 2026. [submetido para revisão]
