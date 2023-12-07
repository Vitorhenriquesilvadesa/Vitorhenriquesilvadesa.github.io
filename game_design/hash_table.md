# Um Pouco Sobre Hash Tables

Uma tabela hash (hash table) é uma estrutura de dados usada para armazenar um conjunto de chave e valor, ou vários valores associados a uma mesma chave, de forma a tentar minimizar o tempo de busca dos dados.

Em uma tabela hash, o processo de armazenamento e recuperação de dados é otimizado para garantir eficiência. A chave é submetida a uma função de hash, que transforma a chave em um índice numérico. Esse índice é utilizado para acessar a posição de armazenamento na tabela hash. Na implementação usada nesse artigo, farei a implementação da tabela hash da seguinte forma. Analogamente a uma tabela desenhada a mão, uma tabela pode ter uma ou mais colunas, e essas colunas podem ter multiplos valores, então teremos diferentes objetos que juntos irão compôr a tabela. Segue um exemplo abaixo:

| Índice | Lista    |
| ------ | -------- |
| 0      | 35       |
| 1      | 12 -> 22 |
| 2      | 8        |
| 3      | 42 -> 3  |
| 4      | 17 -> 27 |

Usaremos uma estrutura que guarde as linhas, ou colunas da tabela (para ver linhas basta girar a tabela em 90 graus), e estruturas que representem essas linhas, da mesma forma, será feita uma classe que conterá dados relacionados a cada elemento da tabela da forma mais didática possível. Além disso, nessa implementação, não permitiremos repetição de valores na tabela.

<br>

## - Função de Hash

A função de hash desempenha um papel crucial no desempenho da tabela hash. Ela mapeia a chave para um índice na tabela. Idealmente, a função de hash deve distribuir as chaves de forma uniforme, minimizando colisões. Colisões ocorrem quando duas chaves diferentes resultam no mesmo índice. Existem diversas técnicas para lidar com colisões, como encadeamento (chaining) e endereçamento aberto.

<br>

## - Vantagens da Utilização de Tabelas Hash

- Velocidade de Acesso O(1): Em condições ideais, a busca de um valor em uma tabela hash tem complexidade O(1), ou seja, é realizada em tempo constante. Isso é possível devido à indexação direta proporcionada pela função de hash.

- Estrutura Dinâmica: As tabelas hash podem crescer dinamicamente à medida que mais elementos são inseridos, adaptando-se à quantidade de dados.

- Eficiência de Espaço: Quando implementadas corretamente, as tabelas hash podem oferecer uma eficiência notável no uso de espaço, pois permitem uma distribuição eficaz dos dados.

Agora falarei um pouco sobre as formas mencionadas anteriormente para tratar colisões em tabelas hash: O Encadeamento; e o Endereçamento Aberto.

<br>

## Encadeamento

A técnica de encadeamento consiste em, sempre que houver uma colisão, inserir o elemento desejado na linha designada para inserção de forma encadeada, por exemplo, o número 32 foi inserido na tabela mostrada anteriormente, repare que ele foi inserido de forma encadeada na linha 1 da tabela:

| Índice | Lista          |
| ------ | -------------- |
| 0      | 35             |
| 1      | 12 -> 22 -> 32 |
| 2      | 8              |
| 3      | 42 -> 3        |
| 4      | 17 -> 27       |

Inserção com Encadeamento: Se ocorre uma colisão, o novo elemento é adicionado à lista existente na posição correspondente.

Busca com Encadeamento: A função de hash é aplicada à chave de busca, o índice é obtido, e então uma busca linear ou outra técnica é usada na lista ligada para encontrar o elemento desejado.

Remoção com Encadeamento: Similar à busca, a posição é encontrada usando a função de hash, e a remoção é realizada na lista associada àquela posição.

O encadeamento é flexível e lida bem com colisões frequentes. No entanto, pode haver algum desperdício de espaço devido à necessidade de armazenar referências para os elementos na lista. Otimizaremos nossa tabela a medida que ela for sendo implementada.

### Vantagens da técnica de encadeamento

- O encadeamento é uma abordagem relativamente simples de implementar. A criação e manipulação de listas ligadas para resolver colisões são conceitos diretos, facilitando o desenvolvimento e manutenção do código.

- O encadeamento permite a alocação dinâmica de memória para as listas ligadas, o que significa que a tabela hash pode crescer ou diminuir conforme necessário. Isso proporciona flexibilidade em relação à quantidade de dados a serem armazenados.

- Em situações onde colisões são frequentes, o encadeamento pode ser uma escolha eficiente. A criação de listas ligadas nas posições da tabela hash permite agrupar elementos com a mesma posição de hash, preservando a estrutura da tabela.

- O encadeamento pode manter um desempenho mais equilibrado mesmo sob diferentes cargas de trabalho, pois a complexidade de busca permanece relativamente estável, independentemente do número de elementos na tabela.

- A gestão de colisões no encadeamento é simplificada. Novos elementos podem ser facilmente adicionados às listas ligadas nas posições da tabela hash, sem a necessidade de reorganizar ou realocar outros elementos.

- À medida que a quantidade de dados aumenta, o encadeamento permite que as listas ligadas cresçam dinamicamente, adaptando-se às mudanças nos padrões de inserção de elementos.

- Em comparação com algumas técnicas de endereçamento aberto, o encadeamento pode ser mais tolerante a cargas altas, pois a estrutura de listas ligadas mantém a organização dos elementos em caso de colisões.

### Desvantagens da técnica de encadeamento

- O encadeamento requer o armazenamento de referencias adicionais para implementar as listas ligadas. Isso pode resultar em um consumo adicional de memória, especialmente quando há muitas colisões.

- Se a tabela hash sofrer colisões frequentes e gerar listas longas, a eficiência da busca pode ser prejudicada. Em casos extremos, as operações podem se aproximar da complexidade de O(n), onde "n" é o número total de elementos na lista ligada.

- O encadeamento pode levar a um uso menos eficiente da cache devido à natureza dispersa das listas ligadas na memória. Isso pode resultar em um desempenho inferior em comparação com outras técnicas que armazenam elementos diretamente na tabela hash.

- A implementação do encadeamento pode ser mais complexa em comparação com outras técnicas de resolução de colisões, especialmente quando se trata de gerenciar dinamicamente o espaço de memória para as listas ligadas.

- O desempenho do encadeamento pode ser sensível ao fator de carga da tabela hash, que é a razão entre o número de elementos na tabela e o tamanho total da tabela. Se o fator de carga aumentar significativamente, as colisões podem se tornar mais frequentes, impactando o desempenho.

<br>

## Endereçamento Aberto

Ao contrário do encadeamento, o endereçamento aberto aborda as colisões de uma maneira diferente. Em vez de utilizar listas ligadas, o endereçamento aberto armazena todos os elementos diretamente na tabela hash. Quando ocorre uma colisão, o endereçamento aberto procura por uma posição alternativa na tabela para armazenar o novo elemento.

Quando é necessário inserir um elemento utilizando o endereçamento aberto e ocorre uma colisão, a função de hash é ajustada para encontrar a próxima posição disponível na tabela. Existem diferentes estratégias para escolher essa nova posição, como avançar linearmente (linear probing) ou avançar quadraticamente (quadratic probing).

Na busca utilizando o endereçamento aberto, a função de hash é aplicada e, se a posição resultante não contiver o elemento desejado, a busca continua seguindo a estratégia de sondagem até encontrar o elemento desejado ou uma posição vazia.

Para a remoção com endereçamento aberto, o processo é semelhante à busca. A posição é encontrada utilizando a função de hash, e o elemento é removido. É crucial marcar as posições que tiveram elementos removidos para garantir a consistência nas operações de busca.

Embora o endereçamento aberto possa ser mais eficiente em termos de espaço, ele requer uma gestão cuidadosa das colisões e pode ser mais sensível à carga da tabela. A escolha entre encadeamento e endereçamento aberto depende das características específicas do problema e dos requisitos de desempenho da aplicação.
