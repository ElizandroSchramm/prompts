# Prompt: Measure Claude Code Token Usage by Skill

Quero instrumentar o desenvolvimento do Vale para medir o consumo real de tokens e o tempo de execução de cada etapa do nosso fluxo Spec-Driven Development com Spec Kit.

**Objetivo:** entender quanto custa executar cada etapa, identificar onde o consumo se concentra e avaliar se o rigor adicional do processo se justifica pelo resultado obtido.

Nosso fluxo atual é:

1. `/speckit-specify`
2. `/speckit-clarify`, quando necessário
3. `/speckit-plan`
4. `/speckit-tasks`
5. `/speckit-analyze` — obrigatório
6. `/speckit-implement`
7. `/speckit-converge`

### 1. Investigue antes de implementar

Inspecione o repositório e o ambiente de desenvolvimento para identificar:

- Como o Spec Kit está instalado e onde estão definidas as skills.
- Qual versão do Claude Code está sendo utilizada e como ela registra o uso de tokens.
- Onde estão armazenadas as transcrições das sessões e quais campos de uso estão disponíveis.
- Como identificar o modelo utilizado em cada execução, inclusive quando houver troca de modelo ou uso de subagentes.
- Quais mecanismos de hooks ou scripts podem ser utilizados sem interferir no fluxo normal de desenvolvimento.

Não presuma que os campos ou caminhos são fixos. Valide-os no ambiente real.

### 2. Implemente a instrumentação

Crie uma solução local, simples e de baixo custo operacional, que registre o início e o fim de cada etapa.

Cada execução deve ter um identificador próprio e registrar, quando disponíveis:

- Etapa do Spec Kit.
- Identificador da feature ou tarefa.
- Data e hora de início e fim.
- Duração total em segundos.
- Modelo ou modelos utilizados.
- Quantidade de chamadas/respostas do modelo.
- `input_tokens`.
- `cache_read_input_tokens`.
- `cache_creation_input_tokens`.
- `output_tokens`.
- Total de tokens processados, calculado e claramente definido como a soma das categorias anteriores.
- Custo estimado em moeda, quando for possível calculá-lo com preços válidos para o modelo e a modalidade de uso.
- Quantidade de chamadas de ferramentas e uso de subagentes, quando disponível.
- Resultado da etapa e eventuais erros.

Use os dados reais fornecidos pelo ambiente. Não peça ao modelo para estimar ou declarar manualmente o próprio consumo.

### 3. Atribuição correta dos tokens

A medição precisa respeitar algumas regras:

- Instrumente os limites das etapas e associe as mensagens das transcrições aos respectivos intervalos.
- Inclua o consumo dos subagentes quando houver dados disponíveis, sem duplicar registros.
- Diferencie tokens de entrada não cacheados, tokens lidos do cache, tokens usados para criar cache e tokens de saída.
- Não confunda tokens processados com tokens faturados ou com a cota consumida no plano.
- Considere que uma mesma sessão pode carregar o histórico anterior em diversas chamadas, e que uma chamada pode atravessar a fronteira entre duas etapas.
- Se a atribuição exata por etapa não for possível, registre essa limitação e apresente a estimativa como estimativa. Não invente precisão.

Mantenha também um total consolidado da sessão, independente da divisão por etapa, para permitir conferir a consistência dos dados.

### 4. Gere um relatório útil

Crie um comando simples para consultar os resultados, por exemplo:

`./scripts/spec-usage report`

O relatório deve apresentar uma tabela por feature e etapa, contendo:

- Duração.
- Tokens de entrada.
- Tokens de cache.
- Tokens de saída.
- Total processado.
- Custo estimado, quando disponível.
- Modelo utilizado.

Também deve permitir comparar features diferentes e identificar as etapas com maior consumo.

Se fizer sentido, gere CSV para análise posterior.

### 5. Preserve o ambiente

- Armazene os dados localmente, sem serviços externos.
- Não registre prompts completos, conteúdo de arquivos, segredos ou dados pessoais desnecessários.
- Não altere o comportamento funcional das skills nem remova verificações obrigatórias.
- Evite adicionar dependências desnecessárias.
- Não comprometa os arquivos de configuração global do Claude Code sem necessidade.

### 6. Validação

Antes de concluir:

1. Execute a solução em uma etapa real do fluxo.
2. Verifique se os registros são produzidos automaticamente.
3. Compare os totais com os dados disponíveis na transcrição original.
4. Verifique como o sistema trata sessões retomadas, etapas interrompidas e subagentes.
5. Documente as limitações e como executar o relatório.

**Primeiro, apresente um diagnóstico breve da arquitetura atual e um plano de implementação. Depois implemente a solução, teste-a e documente como utilizá-la.**