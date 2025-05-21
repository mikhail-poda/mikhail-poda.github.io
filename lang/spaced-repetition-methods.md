# Spaced Repetition Method

[Spaced repetition](https://en.wikipedia.org/wiki/Spaced_repetition) is a learning technique that optimizes review timing to strengthen memory retention. Algorithms like SuperMemo (SM-2), used in apps such as Anki and Duolingo, schedule reviews based on how well you recall an item. Words you struggle with appear more frequently, while familiar ones are shown less often. This approach is based on the spacing effect, which helps transfer information from short-term to long-term memory efficiently.

## Anki's Approach
Anki uses a modified version of the [SuperMemo SM-2 algorithm](https://en.wikipedia.org/wiki/SuperMemo#Description_of_SM-2_algorithm) with a fixed daily card limit structure:

- Fixed Daily Limits: Users predetermine how many new cards they want to learn per day (e.g. 20 new words), and Anki maintains this limit strictly.
- Review Process: When reviewing a card, users select from four response options:
  - Again (complete failure): Card is shown again in < 10 minutes
  - Hard: Card interval increases slightly (1.2× the previous interval)
  - Good: Card interval increases moderately (2.5× the previous interval)
  - Easy: Card interval increases substantially (3.5× the previous interval)
- Interval Calculation: The exact interval depends on the card's history and the user's performance rating.
- Card Scheduling: Cards are scheduled based on their calculated intervals, creating a personalized review queue.

## Card Direction in Spaced Repetition: Optimizing Recognition vs. Production

When learning vocabulary through spaced repetition, an important consideration is card direction: whether to show the word in your target language (L2) and recall its meaning in your native language (L1), or vice versa. Research on vocabulary acquisition reveals distinct cognitive processes for recognition (L2→L1) versus production (L1→L2):

- **Recognition (L2→L1):** When you see a foreign word and recall its meaning, you're building receptive vocabulary that supports **reading** and **listening** comprehension.
- **Production (L1→L2):** When you see a word in your native language and need to produce it in the target language, you're developing productive vocabulary essential for **speaking** and **writing**.

Therefore learning should start as **recognition** and then switch to **production**.

# My Apps
I have created two Flutter apps that use the different flavours of Spaced Repetition Method. They have the same data source from [Google Docs](https://docs.google.com/spreadsheets/d/e/2PACX-1vTTUPG22pCGbrlYULESZ5FFyYTo9jyFGFEBk1Wx41gZiNvkonYcLPypdPGCZzFxTzywU4hCra4Fmx-b/pubhtml)
with vocabulary A0-B2.

## Time-Based Approach
My [first app](https://github.com/mikhail-poda/mila) used a fixed exponential time spacing factor of 1.67, meaning each word had a rank (starting at 1), and the next repetition was scheduled in `1.67^rank` minutes. The rank ranged from 1 to 26, corresponding to repetition intervals from minutes to a year. The recall options ("again," "good," "easy") adjusted the rank accordingly.

However, this method introduced several issues:

**Inflexible Scheduling** – If I studied many new words in one session and then took a break, the system piled up a massive backlog.

**Overwhelming Reviews** – Long gaps between sessions led to a flood of overdue cards, making it difficult to learn new vocabulary.

**Break Sensitivity** – The algorithm treated longer gaps as more significant, meaning a one-week break resulted in a much larger review backlog than a one-day break

### Unexpected Issues with Time-Based Spacing
The above issues led to two unintended consequences:

**Separation of Learning & Review Phases** – The system unintentionally split learning into two disconnected modes:
- A "new vocabulary phase" (learning new words).
- A "review backlog phase" (endless old word reviews, preventing new learning).

**Real-Time vs. Learning-Time Perspective** – The system operated in two different time scales - learning time and real time - each relevant at different stages of the process. Lower word rank values (e.g. repeat in 1 minute) are meaningful within an active study session, in contrast, higher rank values (e.g. repeat in one month) applied in real time, determining long-term retention. However, the system did not distinguish between study gaps and active learning, leading to excessive backlog growth after breaks. Ideally, pause time should be "frozen" so that the scheduling remains based on active study time only, rather than real-world time passing.

## Index-Based Approach

To overcome the limitations of time-based scheduling, my [second app](https://github.com/mikhail-poda/mila_ivrit) adopted a purely index-based system. Unlike the time-based method, which used a fixed word list where new words were introduced sequentially and reviews were scheduled based on overdue time, the index-based approach relies solely on word positioning in a mutable list, with the first element always being the current word under review. This and other changes are described in following sections.

### Scheduling

Instead of scheduling reviews at specific time intervals, the system dynamically repositions words based on recall performance using the formula `new_index = 2^rank`. This means that words you remember well move exponentially further down the list, while harder words stay closer to the beginning.

One notable aspect of this system is how it handles vocabulary across proficiency levels. For example, if a beginner (A1) word reaches a rank of 10, the exponential formula (2^10 = 1024) places it at position 1024 in the list. This works best when the vocabulary list is large enough to include more advanced words (B1 and higher).

### Card Direction

A progressive approach to card direction is implemented to insure that learners aren't overwhelmed with production tasks before recognition is solid:
- Ranks 0-6: Words are presented exclusively in L2→L1 direction (**recognition mode**)
- Rank 7+: Words begin appearing in L1→L2 direction (**production mode**)

### Rank Calculation

While the recall options are still displayed as ("again," "good," "easy"), they define a scale where user selection (touch) is evaluated as a float number between minimum and maximum rank (1-11). The new rank is calculated using an adaptive algorithm:
- If the selected rank is smaller than the old rank, the new rank becomes exactly the selected rank (allowing for immediate demotion when words are forgotten)
- If the selected rank is greater than the old rank, the new rank becomes the mean value of the selected rank and the old rank (creating a gradual promotion system for words being learned)

### Advantages
The index-based scheduling method eliminates overdue backlogs, seamlessly integrates learning and review, and adapts dynamically to difficulty. Most importantly, it remains completely unaffected by breaks in study - whether you pause for a day or a month, the system continues exactly where you left off. Unlike time-based systems, it ensures that learning stays productive without being disrupted by long gaps or review overload.

## Lessons Learned: Simplicity Improves Learning
Another key insight from using both apps was that displaying less information led to better learning outcomes. Initially, I included extra details such as synonyms and related root words, assuming they would reinforce memory. However, I found that these additional elements often distracted from active recall and the quick assessment of word knowledge. By simplifying the learning process, throughput increased at least twofold, while the cognitive load for each new word decreased, making learning more efficient and less mentally taxing.
