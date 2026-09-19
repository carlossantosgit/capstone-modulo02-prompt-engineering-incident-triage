# Capstone Módulo 02 — Sistema de Prompt Engineering para Aplicações Empresariais

**Intelligent Incident Triage & Response System** (Sistema Inteligente de Triagem e Resposta a Incidentes) — documento principal: relatório final de avaliação.

**Versão em PDF:** [Relatorio-Final-Capstone-Modulo02.pdf](Relatorio-Final-Capstone-Modulo02.pdf)

> **Nota sobre este repositório.** Este repositório público contém apenas o documento principal. O projeto completo (8 prompts V1/V2, schemas, suíte de 12 casos, scripts de métricas, saídas brutas das 72 execuções do pipeline e das 60 execuções do validador, exemplos e demais documentos) é referenciado abaixo pelos caminhos relativos (`docs/…`, `prompts/…`, `tests/…`, `results/…`) e é entregue separadamente (arquivo .zip). Todos os números deste documento vêm de `results/metrics_v1.json` e `results/metrics_v2.json` do projeto completo. Nomes de arquivos, códigos de teste (TC-xxx) e valores de campos (`CORRIGIR`, `SUCCESS`, etc.) foram mantidos como no projeto.

---

# Relatório Final de Avaliação

## Resumo Executivo

Foi projetado, testado e melhorado um pipeline de quatro prompts para **triagem e resposta inteligente a incidentes**. A V1 (linha de base) e a V2 (melhorada) foram executadas nos mesmos 12 casos de teste, em 3 rodadas (**72 execuções completas do pipeline**), além de um teste do validador com defeito único (**60 execuções apenas do Prompt 04**; 78 execuções de um desenho anterior, contaminado e com múltiplos defeitos, foram mantidas como legado) e uma passada de juiz independente para afirmações não sustentadas. Os resultados são reais, produzidos por sub-agentes LLM e pontuados por `tests/evaluate.py`; temperatura e demais parâmetros de amostragem *não eram configuráveis no ambiente utilizado*.

- Sucesso nas tarefas **86,1% → 94,4%** (31/36 → 34/36, +8,3 pp); Nota Geral de Qualidade **90,9% → 97,6%** (+6,7 pp).
- As maiores diferenças estão em consistência e especificação (consistência 80,6% → 91,7%, coerência de status 58,3% → 100%, estabilidade do `incident_id` 0/12 → 12/12, validade sob enums estritos das saídas do validador 87,2% → 100%), acurácia de fallback 91,7% → 100% e detecção direcionada do validador em fixtures de defeito único 76,2% → 100% (16/21 → 21/21).
- **Não melhorou / piorou:** acurácia de categoria 100% → 97,2% (−2,8 pp; regressão no TC-007), consistência de categoria 97,2% → 94,4%; conformidade de schema e tratamento de ambiguidade já estavam em 100%; uma afirmação não sustentada ainda chegou à saída da V2.
- **A confiança no resultado é limitada:** a diferença de sucesso nas tarefas é de 3 execuções e os intervalos de 95% se sobrepõem (71,3–93,9% contra 81,9–98,5%). A linha de base já era forte (efeito teto) e a suíte é pequena.

## Problema

A triagem de relatos de incidentes não estruturados deve ser padronizada, apoiada em evidências e nunca fabricar fatos ou causas; deve passar o caso a humanos quando a informação é insuficiente. Ver `docs/01_business_problem_and_scope.md` (não existia problema pré-definido para este capstone na pasta do módulo, então foi usado o cenário alternativo do enunciado).

## Arquitetura

P1 Análise e Normalização da Entrada → P2 Classificação e Análise de Contexto → P3 Decisão e Recomendação → P4 Validação e Resposta Final, com um orquestrador que fornece metadados confiáveis, encaminha entrada bloqueada direto ao estágio 4 e repete o estágio 3 uma vez quando o veredito é `CORRIGIR`. Quatro JSON Schemas definem os contratos (congelados entre as versões). Diagrama e tabelas de contrato: `docs/02_architecture_and_orchestration.md`. A fonte externa é **simulada** (observações embutidas na entrada de teste); o orquestrador foi executado pelo agente de teste, não por código.

## Sistema de Prompts

Cada prompt tem seções de papel/responsabilidade, entrada/contexto, instruções/restrições e contrato de saída; a V2 acrescenta hierarquia de instruções, um exemplo trabalhado e regras derivadas das evidências. Técnicas (prompting estruturado, encadeamento, raciocínio estruturado por campos auditáveis, protocolo de campos inspirado em ReAct, few-shot, prompting por restrições, validador de autoverificação, imposição de schema), onde e por quê: `docs/03_prompt_design_rationale.md`. Arquivos: `prompts/0X_*/v1.md` e `v2.md` (8 prompts).

## Metodologia de Teste

Rubrica fixada antes de qualquer saída ser inspecionada; 3 rodadas independentes por versão; executores cegos (proibidos de ler o comportamento esperado); juiz independente para afirmações não sustentadas; fixtures de defeito único para o validador (7 falhas + 3 controles limpos, válidos nos dois contratos), porque as execuções naturais tiveram poucos erros a montante para medir o validador. Registro de reprodutibilidade, ressalvas de execução e ameaças à validade: `docs/05_evaluation_methodology.md`. Duas correções de rubrica/fixtures feitas após a V1 (aplicadas às duas versões) estão registradas em `tests/evaluation_rubric.md` §6. Ressalvas principais: dentro de uma rodada os casos foram produzidos em um único contexto de agente; executor e juiz são da mesma família de modelos.

## Cobertura de Testes

12 casos: simples (TC-001, 002), complexos (003, 004), ambíguos (005, 006), incompleto (007), contraditório (008), dependente de fonte externa (009), falha de fonte externa (010), adversarial/prompt injection (011), fallback obrigatório (012; também 005, 007, 010). Comportamento esperado por caso: `tests/expected_behaviors.md`; versão legível por máquina: `tests/test_cases.yaml`.

## Métricas

Dez métricas obrigatórias mais três diagnósticos, definidas em `tests/evaluation_rubric.md`. Nota Geral = 0,20·Sucesso + 0,15·Classificação + 0,10·Schema + 0,10·(1−Alucinação) + 0,10·Fallback + 0,10·Ambiguidade + 0,10·Consistência + 0,075·Detecção + 0,075·Ancoragem (pesos somam 1,0).

## Resultados da V1

| Métrica | V1 |
|---|--:|
| Taxa de sucesso nas tarefas | 86,1% (31/36) |
| Conformidade de schema | 100% (144/144 saídas de estágio) |
| Acurácia de classificação | 100% (categoria 36/36, severidade 36/36) |
| Taxa de alucinação / afirmações não sustentadas | 5,6% (2/36) |
| Acurácia de fallback | 91,7% (22/24) — 0 ausentes, 2 excessivos |
| Tratamento de ambiguidade | 100% (9/9) |
| Consistência entre rodadas | 80,6% |
| Taxa de detecção do validador (defeito único, detecção direcionada) | 76,2% (16/21); falsos positivos 0% (0/9) |
| Validade sob enums estritos das saídas do validador | 87,2% (34/39) |
| Ancoragem em evidência externa | 83,3% (5/6) |
| Nota Geral de Qualidade | 90,9% |
| Diagnósticos | coerência de status 58,3%; consistência do incident-id 0% |

## Padrões de Erro

Tabela completa com frequências, causas e correções: `results/error_patterns.md`. Principais padrões da V1: E09 saída inconsistente (instabilidade do ID em 12/12 casos; semântica sobrecarregada de `escalation_required` → 15/36 execuções incoerentes; variação de veredito/status entre rodadas), E02 afirmações causais não sustentadas (2/36), E06 escalonamento excessivo no caso de injeção (2/24), E10 pontos cegos do validador (0/2 vazamentos naturais detectados; VF-05 não detectado 3/3; 5/39 rótulos em texto livre). E01, E05, E07 e E08 foram observados 0 vezes.

## Melhorias Implementadas

Dez mudanças C1–C10, cada uma com problema → evidência → hipótese → mudança → resultado, em `CHANGELOG.md`. Essência: `incident_id` fornecido pelo orquestrador; `escalation_required ⇔ HUMAN_REVIEW_REQUIRED`, tabela determinística de status e mapeamento de status; rubrica de severidade e faixas de confiança; campos de evidência externa no estilo ReAct; mitigação condicional; validador recebendo o relato bruto com checklist C1–C8; roteador e laço de uma repetição; um exemplo few-shot por prompt.

## Resultados da V2

| Métrica | V2 |
|---|--:|
| Taxa de sucesso nas tarefas | 94,4% (34/36) |
| Conformidade de schema | 100% (138/138 saídas de estágio; 6 puladas pelo roteador) |
| Acurácia de classificação | 98,6% (categoria 35/36, severidade 36/36) |
| Taxa de alucinação / afirmações não sustentadas | 2,8% (1/36) |
| Acurácia de fallback | 100% (24/24) |
| Tratamento de ambiguidade | 100% (9/9) |
| Consistência entre rodadas | 91,7% |
| Taxa de detecção do validador (defeito único, detecção direcionada) | 100% (21/21); falsos positivos 0% (0/9) |
| Validade sob enums estritos das saídas do validador | 100% (39/39) |
| Ancoragem em evidência externa | 100% (6/6) |
| Nota Geral de Qualidade | 97,6% |
| Diagnósticos | coerência de status 100%; consistência do incident-id 100%; taxa de repetição 0% |

## Comparação V1 × V2

| Métrica | V1 | V2 | Δ pp | Δ rel |
|---|--:|--:|--:|--:|
| Sucesso nas tarefas | 86,1% | 94,4% | +8,3 | +9,7% |
| Conformidade de schema | 100,0% | 100,0% | 0,0 | 0,0% |
| Acurácia de classificação | 100,0% | 98,6% | −1,4 | −1,4% |
| Taxa de alucinação (↓) | 5,6% | 2,8% | −2,8 | −50,0% |
| Acurácia de fallback | 91,7% | 100,0% | +8,3 | +9,1% |
| Tratamento de ambiguidade | 100,0% | 100,0% | 0,0 | 0,0% |
| Consistência | 80,6% | 91,7% | +11,1 | +13,8% |
| Detecção do validador (defeito único, direcionada) | 76,2% | 100,0% | +23,8 | +31,3% |
| Validade sob enums estritos, saídas do validador | 87,2% | 100,0% | +12,8 | +14,7% |
| Ancoragem | 83,3% | 100,0% | +16,7 | +20,0% |
| Nota Geral de Qualidade | 90,9% | 97,6% | +6,7 | +7,4% |

Tabelas por caso, por fixture e de sensibilidade aos pesos: `tests/comparison.md`.

## Melhoria Medida

```
Sucesso nas tarefas:      V1 = 86,1%  V2 = 94,4%  Melhoria = +8,3 pontos percentuais (31/36 → 34/36)
Consistência:             V1 = 80,6%  V2 = 91,7%  Melhoria = +11,1 pontos percentuais
Coerência de status:      V1 = 58,3%  V2 = 100%   Melhoria = +41,7 pontos percentuais (diagnóstico)
Detecção do validador:    V1 = 76,2%  V2 = 100%   Melhoria = +23,8 pontos percentuais (16/21 → 21/21; 7 fixtures de defeito único × 3 rodadas)
Nota geral:               V1 = 90,9%  V2 = 97,6%  Melhoria = +6,7 pontos percentuais
Acurácia de categoria:    V1 = 100%   V2 = 97,2%  Variação = −2,8 pontos percentuais (regressão, TC-007 r3)
```

Por caso: TC-011 1/3 → 3/3, TC-004 2/3 → 3/3, TC-009 2/3 → 3/3, TC-007 3/3 → 2/3 (regressão), demais inalterados. A diferença da Nota Geral permanece entre +4,3 e +7,7 pp sob quatro esquemas de pesos diferentes. **Ressalva estatística:** os intervalos do sucesso nas tarefas se sobrepõem; trate como evidência direcional. Os ganhos mais fortes e menos sensíveis a ruído são os determinísticos (estabilidade do ID, coerência de estado); os que um revisor deve confiar menos são a taxa de alucinação (1 contra 2 eventos) e os números de detecção do validador (7 fixtures × 3 rodadas, padrões de evidência frouxos, validador simulado por LLM, casos de uma rodada produzidos em um único contexto).

Verificações pontuais feitas (não é anotação humana): os motivos dos bloqueios do validador V2 em VF-01, VF-02, VF-05 e VF-08 (rodada 1) foram lidos e correspondem à falha injetada; os registros marcados pelo juiz (V1: TC-003 r2, TC-009 r2; V2: TC-003 r2) e a saída do TC-007 r3 foram lidos e são coerentes com os rótulos.

## Correções pós-revisão

Duas revisões adversariais independentes (Codex) questionaram os resultados. Cada afirmação foi conferida contra os dados (`results/raw/`, `tests/`) antes de agir.

**Rodada 1**

| Afirmação | Verificada? | Ação tomada |
|---|---|---|
| O ganho de detecção vinha só do VF-05, uma regra que só a V2 define | **Sim** | Bundles de múltiplos defeitos rebaixados a legado; ver rodada 2 |
| Registros ausentes do juiz contavam silenciosamente como "sem alucinação" | Defeito de código confirmado; sem efeito nos números publicados | Carga do juiz tornada estrita (versão completa na rodada 2) |
| Qualquer valor diferente de `APROVAR` contava como detecção | Confirmado no código; nenhum veredito inválido ocorreu | Um bloqueio exige saída válida no schema e veredito de bloqueio reconhecido |
| `final_category`/`final_severity` aceitam qualquer string | Confirmado | `schemas/final_output.strict.schema.json`; validade estrita: saídas finais do pipeline 36/36 nas duas versões, saídas do validador 34/39 (V1) contra 39/39 (V2). Schema original inalterado |
| `[EXTERNAL]` confia em afirmações dentro do relato não confiável | Confirmado por desenho | Documentado como limitação de segurança; não corrigido, não testado |
| O resumo final é escrito depois da auditoria C2 e não é auditado | Confirmado (TC-003 r2, V2) | Documentado; correção da V3 listada |

**Rodada 2**

| Afirmação | Verificada? | Ação tomada |
|---|---|---|
| A checagem "estrita" do juiz ainda aceita null / campos externos ausentes | **Sim** — só a presença das chaves era verificada | Registros precisam ter listas de strings em `*_unsupported` e booleanos nos campos do TC-009/TC-010; `tests/test_evaluate_guards.py` comprova que registros nulos, ausentes, com tipo errado, duplicados ou faltando interrompem a pontuação. Os arquivos do juiz já publicados cumpriam a regra, então nenhum número mudou |
| Excluir o VF-05 não remove a contaminação dos outros bundles | **Sim** — VF-01 e VF-02 têm `escalation_required=true` com `SUCCESS`, VF-08 tem `true` com `NEEDS_INFORMATION`, e o VC-01 sem modificação é bloqueado pela V2 sem defeito injetado | Substituídos por **fixtures de defeito único** válidos nos dois contratos (7 falhas, 3 controles limpos), executados para V1 e V2 × 3 rodadas (60 execuções reais do validador), exigindo que as notas citem a falha injetada. Resultado: V1 16/21 (76,2%), V2 21/21; 0/9 falsos positivos nas duas. Erros da V1: evidência externa inventada (aprovada 2/3) e contradição removida (aprovada 3/3). Nota geral recalculada: V1 90,9%, V2 97,6% |
| As conclusões atribuem confiabilidade ao pipeline sem execução isolada nem avaliação reservada | **Sim** (já declarado como limitação) | Conclusões reescritas para descrever diferenças nos artefatos desta simulação conjunta; sem afirmações causais por componente |

Ainda não feito (exige novas execuções ou trabalho humano): reexecutar cada estágio em contexto separado apenas com entradas contratuais, suíte reservada (held-out), revisão humana cega dos rótulos que decidem sucesso e alucinação e testes de fonte forjada (`[EXTERNAL]`). A checagem de padrão de evidência para a "detecção direcionada" é frouxa por desenho; li as notas de vários fixtures (V2 SF-02/SF-04/SF-06, V1 SF-02/SF-06) e correspondiam às falhas injetadas.

## Limitações Restantes

1. **Suíte pequena e relativamente fácil.** 12 casos e 3 rodadas; a V1 forte deixa pouca margem; casos limítrofes aceitam vários rótulos, então as métricas de acurácia discriminam pouco.
2. **Fidelidade da execução.** Sub-agentes LLM simulam os estágios; casos de uma rodada compartilham contexto; sem parâmetros de amostragem; executor e juiz são da mesma família de modelos; sem rótulos humanos de referência.
3. **Fonte externa e orquestrador simulados.** Sem laço real de ferramenta; a validação de schema com repetição e o laço de repetição nunca foram acionados em execuções naturais (taxa de repetição 0%). Formato de entrada inválido, informação irrelevante e saída de estágio que não é JSON são *apenas desenho*.
4. **Os diagnósticos codificam a semântica da V2** (coerência), e a rubrica de severidade da V2 foi escrita junto com os rótulos esperados.
5. **Defeitos da V2 deixados sem correção para manter a comparação válida:** categoria em relatos vagos, limite da confiança, redação do SEV1 para janelas de acesso já encerradas, redação causal do resumo, regra de rótulo para `SKIPPED_BY_ROUTER` (`CHANGELOG.md`, "V2 residual gaps"). Enums de schema para os campos `final_*` não adicionados ao schema original.
6. **O validador em execuções naturais não está provado confiável:** aprovou uma categoria errada e seu próprio resumo trouxe uma afirmação causal não sustentada; sua vantagem medida em fixtures de defeito único (V1 76,2% contra V2 100%) é evidência de amostra pequena.
7. **Confiança em fonte externa.** Fatos `[EXTERNAL]` vêm do próprio relato não confiável; um remetente pode fabricar uma fonte "consultada". Não testado (sem casos de falsificação).
8. **A evidência sobre o validador é fina:** 7 fixtures de defeito único × 3 rodadas, padrões de evidência frouxos, fixtures escritos pelos mesmos autores e da mesma família de modelo do validador, e os 10 itens de cada rodada foram produzidos em um único contexto.
9. **Sem medição de custo, latência ou tokens.**

## Considerações para Produção

Não está pronto para produção: sem código de orquestração, logs, redação de dados sensíveis, monitoramento ou fluxo de revisão humana; resistência a injeção medida em um caso e um fixture; parâmetros de amostragem não fixados. Passos necessários e análise de lacunas em `docs/07_production_considerations.md` (piloto em modo sombra, camada de redação, suíte rotulada maior, portão de promoção sobre estas métricas, modelo e parâmetros fixados).

## Conclusão

Dentro desta simulação, os artefatos da V2 são mais consistentes e mais explicitamente especificados que os da V1: identificadores estáveis, campos de status coerentes, consistência de veredito de 100%, sem placeholders improvisados em rótulos, fallback correto em todas as 24 execuções avaliáveis e um validador que bloqueou e citou corretamente a falha em todas as 21 execuções de defeito único (V1: 16/21), sem falsos positivos. Também introduziu uma pequena regressão de classificação (TC-007) e ainda deixou passar uma afirmação não sustentada. Essas são diferenças entre as saídas de uma simulação conjunta por LLM, executada uma vez por rodada em contextos compartilhados, ajustada sobre os mesmos 12 casos e julgada por um LLM da mesma família; elas **não** mostram que cada mudança de prompt causou as diferenças, nem que o pipeline se comportaria assim com estágios isolados, suíte reservada ou resultados rotulados por humanos. O valor de engenharia do exercício está tanto no *método* — linha de base congelada, execuções cegas, injeção de falhas, correções registradas e duas revisões que cada uma derrubou parte de uma conclusão anterior — quanto nos números. A próxima iteração deve corrigir as lacunas documentadas da V2, adicionar enums ao schema, rodar os estágios isolados numa suíte reservada e rotulada por humanos, incluir testes de fonte forjada e construir um orquestrador real com laço de ferramenta externa ao vivo.

## Referências

- Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., Chi, E., Le, Q., & Zhou, D. (2022). *Chain-of-Thought Prompting Elicits Reasoning in Large Language Models*. arXiv:2201.11903. https://arxiv.org/abs/2201.11903
- Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2022). *ReAct: Synergizing Reasoning and Acting in Language Models*. arXiv:2210.03629. https://arxiv.org/abs/2210.03629
- DAIR.AI. *Prompt Engineering Guide*. https://www.promptingguide.ai/ (acessado em 2026-09-19).
- OpenAI. *OpenAI Platform Documentation*. https://platform.openai.com/docs (referência fornecida pelo curso; não consultada neste trabalho).
