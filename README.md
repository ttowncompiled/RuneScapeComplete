# RuneScapeComplete
A genetic algorithm to determine the most efficient way to "complete" RuneScape

## Outline
1. Research and collect data.
  * Data includes quests (and miniquests), the necessary tasks to complete each quest, the necessary requirements to start each quest, achievements, the necessary requirements to perform each achievement, and the location of each task (represented as a (x,y,z) coordinate, where x is the number of squares along the East-West axis, y is the number of squares along the North-South axis, and z is the number of levels up or down, using Lumbridge Home Teleport as (0,0,0)).
  * To fully complete a character, a player needs to train to level 99 in all skills and complete all quests and achievements.
  * This guide assumes the the player is a member but does not assume that the player will purchase any additional benefits. A player can improve their efficiency by paying for bonds, which they can then sell for millions in-game, but it will be left to the player to decide if and when they will do so.
  * This guide assumes that the player (essentially) skips tutorial island and begins the guide as a fresh character.
2. Curate a set of directed acyclic graphcs (DAGs).
  * Each quest can be represented as a set of requirements, a set of tasks, and a set of rewards. The requirements must be met before any of the tasks can be attempted. The tasks themselves have a particular order that they must be peformed in. The rewards are only received after the last task is completed. This structure mirrors that of a DAG
  * Each quest has a single source and a single sink. The single source is the quest start, where the player receives the quest from the quest giver. The sink is the quest completion, where the player turns in the quest and receives their reward. This type of DAG is known as a ST-DAG or a Two-Terminal DAG.
  * Achievements are represented as a single node. A single node is still technically a DAG.
3. Use the DAGs to generate a POPULATION.
  * The population has a select size P (input parameter).
  * Each member of the population is created by randomly combining the input DAGs. The input DAGs will also be merged from left to right in the order that they are provided.
  * Let there be two DAGs d_1 and d_2. These DAGs are combined by randomly inserting the members of d_2 before or after members of d_1 such that the members of d_2 maintain their relative order.
  * It is always assumed that skill requirements can be met by skilling before attempting the task.
  * It is always assumed that item requirements can be met by either creating the item or purchasing the item before attempting the task.
4. Calculate the FITNESS of each member of the population.
  * The fitness of each member of the population is based on the cardinal/ordinal distances between consecutive tasks. Cardinal/ordinal distances is grid based movement that includes the use of diagonals, where diagonal movement is only counted as a single space of moverment. It is therefore far more efficient to move diagonally than it is to move solely along the x- or y-axis.
  * The fitness function does not consider natural barriers, such as walls or oceans.
  * The fitness function does not consider time required to skill or create/purchase items (at this time).
5. Select PARENTS from the population using a SELECTOR.
  * We will compare two selectors. The selectors are roulette and tournament.
6. Generate OFFSPRING from the parents using a CROSSOVER OPERATOR.
  * We will compare two crossover operators. The two operators are the modified order-based crossover (MOC) operator and a custom version of the order-based crossover (OBX) operator.
  * We must use a custom version of the OBX operator because the standard OBX operator is invalid for DAGs. For example, let us randomly select two crossover points a_1 and a_2 where a_1 appears before a_2 in parent p_1. Let us also select crossover points b_1 and b_2 from parent p_2 such that a_1 = b_2 and a_2 = b_1 and b_1 appears before b_2 in parent p_2. Let it be that the number of open positions (not a_1) that appear before a_2 is k_a and that the number of open positions (not b_1) that appear before b_2 is k_b. If it is the case that k_b > k_a, then k_b - k_a alleles from parent p_2 that appear before b_2 must be assigned after a_2 in the first offspring of p_1 and p_2. Since these alleles appear before b_2, the task at b_2 might be dependent on one of those alleles which appears after a_2 in the first offspring. In such cases, the relative order of the DAG associated with b_2 would be violated. In addition, since the order of a_1 and a_2 in p_1 is different than their order in p_2, it could also be the case that tasks that must come before a_1 come after a_1 in the first offspring of p_1 and p_2.
  * The custom version of the OBX operator adds alleles from the second parent using an order-based insert method. This insert method checks the selected crossover points before each insertion to determine if the insertion would cause a violation. If there would be a violation, then the allele is inserted in the next previous position that would not cause a violation.
7. Mutate each member of the offspring with probability m using a MUTATION OPERATOR.
  * Each member is mutated with probability m (input parameter).
  * The chosen mutation operator is the insertion mutation operator using the same order-based insertion operator used for the custom OBX operator with a randomly selected allele.
8. Recompute FITNESS and maintain member with the greatest observed fitness from across the generations.
9. Repeat for a select number of GENERATIONS.
