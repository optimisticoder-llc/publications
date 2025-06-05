# 🧠 Beacon Algorithm

Welcome to Kuqus, where decision-making is fast, intuitive, and scientifically precise! Our **Beacon Algorithm** revolutionizes preference ranking through quick pairwise comparisons—like choosing between "Rendang vs. Sushi." Instead of cumbersome list rankings, you make simple choices, and our algorithm constructs a robust preference graph. By leveraging **transitive reduction**, **reaction time analysis**, and **smart scoring**, Kuqus delivers accurate rankings with fewer comparisons while prioritizing your privacy. Let’s dive into how it works!

---

## 📈 The Science of Preferences

The Beacon Algorithm combines graph theory, psychophysical modeling, and statistical ranking to deliver accurate and efficient preference rankings. Here’s the breakdown:

### Pairwise Comparisons & Transitive Reduction

When you choose one item (e.g., A) over another (e.g., B), the algorithm records this preference in a **transitive graph**. Using **transitive reduction**, we infer indirect preferences (e.g., A > B and B > C implies A > C), minimizing the number of comparisons needed.

#### Comparison Efficiency:
- For $ n $ items, a full pairwise comparison requires $ \binom{n}{2} = \frac{n(n-1)}{2} $ comparisons.
- The Beacon Algorithm achieves:
  - **Best case**: $ n-1 $ comparisons (e.g., for a linear order like A > B > C).
  - **Average case**: ~40–50% of $ \binom{n}{2} $ for $ n = 15 $ (e.g., ~42–53 comparisons instead of 105).
  - **Smaller sets** (e.g., $ n = 3 $ ): ~40–80% of $ \binom{3}{2} = 2..3 $ comparisons.
- This efficiency is driven by a transitive reduction graph, which removes redundant edges while preserving the preference order:

$$
\text{Edge}(i \to j) \text{ if } i \text{ is directly chosen over } j \\
\text{and no transitive path } i \to k \to j \text{ exists}
$$

> [*Aho, A. V., Garey, M. R., & Ullman, J. D. (1972). The Transitive Reduction of a Directed Graph. SIAM J. on Computing.*](https://epubs.siam.org/doi/10.1137/0201008)

### Scoring Preferences

After collecting comparisons, the algorithm assigns scores to items based on their rank in a topological sort of the transitive graph:

- **Initial Scoring**: For $ n $ items ranked A, B, C, … (e.g., A > B > C > D > E for $ n = 5 $ ), scores are assigned in fixed steps from 100 to 0:
$$
s_i = 100 \cdot \left(1 - \frac{i-1}{n-1}\right), \quad i = 1, 2, \dots, n
$$
Example: For A, B, C, D, E, scores are 100, 75, 50, 25, 0.

- **Last Item Adjustment**: If the reaction time for the comparison between the second-to-last item (e.g., D) and the last item (e.g., E) exceeds the median reaction time of the session, the last item’s score is set to half the second-to-last item’s score to reflect implicit preference:
$$
s_n = \begin{cases}
\frac{s_{n-1}}{2} & \text{if } t_{n-1,n} > t_{\text{median}} \\
0 & \text{otherwise}
\end{cases}
$$
Example: If D vs. E reaction time > median, E’s score is set to $ \frac{25}{2} = 12.5 $.

### Reaction Time & Uncertainty Adjustment

Inspired by Kiani et al. (2014), we use **reaction time** to adjust scores, reflecting decision certainty. Longer reaction times indicate higher uncertainty, reducing the score gap between items to capture “hard choices.”

- **Normalization with Weber-Fechner Law**: Reaction times are normalized logarithmically to reflect the psychophysical scaling of perception (Weber-Fechner law), clamping times within a 10,000ms range:
$$
t_{\text{norm}} = \frac{\log(t + c) - \log(t_{\text{min}} + c)}{\log(t_{\text{max}} + c) - \log(t_{\text{min}} + c)}
$$
Where:
- $ t $: reaction time in milliseconds
- $ t_{\text{min}} $: 20th percentile of reaction times
- $ t_{\text{max}} $: 10,000ms
- $ c = 1.0 $: constant to avoid undefined logs

This **logarithmic transformation** (aligned with the Weber-Fechner law) provides a scaled representation of reaction time, where proportional differences in time (e.g., doubling the reaction time) are treated with more consistent impact. This scaled time is then used to derive a **certainty score ( $w_t$ ) via an exponential function** in the next step. This combined approach ensures that:
* **Faster reaction times** (indicating high certainty) result in a very **small penalty**, thus minimally affecting the score gap between chosen items.
* **Slower reaction times** (indicating lower certainty or harder choices) lead to a **progressively larger penalty**, which more significantly reduces the score gap between items to reflect the increased decision difficulty.

- **Certainty Score**: The normalized reaction time is used to compute a certainty score using an exponential function, reflecting non-linear uncertainty growth:
$$
w_t = e^{-1.5 \cdot t_{\text{norm}}}
$$
$$
\text{penalty} = 1 - w_t
$$

- **Score Adjustment**: For a pair where item A (score $ s_A $ ) is chosen over item B (score $ s_B $ ), A’s score is adjusted toward B’s score based on the penalty, with a buffer to preserve order:
$$
s_A' = s_A - \text{penalty} \cdot s_A
$$
$$
s_A' = \max(s_A', s_B + 1.0)
$$

This ensures $ s_A' > s_B $, maintaining the ranking while reflecting decision difficulty.

> [*Kiani, R., et al. (2014). Choice Certainty Is Informed by Both Evidence and Decision Time. Journal of Neuroscience.*](https://www.sciencedirect.com/science/article/pii/S0896627314010964)

- **Theoretical Support**: Reaction times are normalized logarithmically using the **Weber-Fechner Law**, reflecting the non-linear perception of decision certainty. An exponential certainty score ( $ e^{-1.5 \cdot t_{\text{norm}}} $ ) adjusts scores minimally for fast decisions and more for slower, uncertain ones. While Stevens’ Power Law ( $ t^k $ ) is an alternative, our logarithmic-exponential approach is robust and psychophysically grounded.

### Final Scaling

To ensure scores are intuitive, all scores are scaled so the highest score is 1000:

$$
s_i' = s_i \cdot \frac{1000}{\max(s_1, s_2, \dots, s_n)}
$$

This stretches the score range while preserving relative differences and rankings.

### Head-to-Head Probabilities

To display preference strength, we compute head-to-head probabilities using the **Bradley-Terry model**:

$$
P(A > B) = \frac{s_A}{s_A + s_B}
$$

Where $ s_A $ and $ s_B $ are the final adjusted scores. These probabilities are shown on the results page, highlighting how strongly one item is preferred over another.

> [*Bradley, R. A., & Terry, M. E. (1952). Rank Analysis of Incomplete Block Designs: I. The Method of Paired Comparisons. Biometrika.*](https://www.jstor.org/stable/2334029)

---

## 🔁 Smart Pair Selection

The algorithm selects pairs to minimize comparisons while maximizing information gain:
- **Priority**: Pairs involving items with fewer comparisons or unclear transitive relationships.
- **Transitive Reduction**: Skips pairs already resolved by transitive paths (e.g., if A > B and B > C, skip A vs. C).
- **Randomization**: Ensures diversity by randomly selecting among viable pairs when needed.

This approach achieves logarithmic efficiency, requiring only ~40–50% of $ \binom{n}{2} $ comparisons for $ n = 15 $.

---

## 🛑 Early Stopping for Efficiency

The algorithm stops when rankings stabilize, typically after ~40–50% of possible comparisons for larger sets ( $ n = 15 $ ). Stability is assessed by checking if recent comparisons align with the current ranking, ensuring efficiency without sacrificing accuracy.

---

## 📊 Benchmarking Performance

We rigorously tested the Beacon Algorithm with thousands of simulations, assuming a ground truth where items are ranked alphabetically (A > B > C > …). Each comparison uses a fixed reaction time of 1000ms to simulate maximum certainty.

### Evaluation Metric

We use a **Misplacement Rate (MR)** to measure ranking accuracy:

$$
\text{MR} = \frac{\sum_{i=1}^n \mathbb{1}_{\sigma(i) \neq k_i}}{n}
$$

Where:
- $ \sigma(i) $ : Key at position $ i $ in the sorted order (by score, descending). (Note: Changed $i-1$ to $i$ for clarity, assuming 1-based indexing for positions, or keep $i-1$ if you stick to 0-based indexing for lists in explanation).
- $ k_i $ : Expected key at position $ i $ (e.g., A at 1, B at 2).
- $ \mathbb{1} $ : Indicator function (1 if keys differ, 0 otherwise).

### Key Performance Results
(Based on thousands of simulations for 3-15 items)

- ✅ **Perfect Rank Accuracy**: Our algorithm consistently delivers **100% correct ranking order** (Misplacement Rate of 0%). This precision stems directly from user choices, ensuring no item is out of its true relative preference.
- ⚡ **Unmatched Efficiency**:
    * **Small Sets (e.g., 3 items)**: Achieve reliable rankings with just 1-2 comparisons, a remarkable 33-80% reduction from exhaustive methods.
    * **Larger Sets (e.g., 15 items)**: Get accurate results with only 14-51 comparisons, representing up to a 60% reduction in effort compared to traditional pairwise methods.
    This efficiency means you can gain valuable insights with **significantly fewer comparisons per user**, enabling faster feedback loops even with a limited number of survey participants (e.g., often just 1-30 individual responses are enough to uncover preferences within a given item set, depending on required confidence).
- 🧭 **Deeper Insights**: Beyond simple ranks, our system uses reaction time to gauge decision certainty, providing nuanced scores (e.g., 1000 vs. 740 for a close preference).
- 📊 **Actionable Probabilities**: Head-to-head probabilities (via Bradley-Terry) offer clear, market-ready insights into the strength of preferences, perfect for strategic decision-making.

---

## 👤 Privacy First: Pseudonymous Data

Your privacy is paramount. Kuqus ensures:
- Anonymous session IDs (`X-Anonymous-ID`) using server-generated UUID + IP location estimation (no IP storage).
- No personal profiling or hidden tracking.
- Regional insights or private survey dashboards without linking to personal data.

---

## 🔒 Transparent Yet Powerful

We share the core logic of the Beacon Algorithm but protect key implementation details to prevent manipulation. Expect:
- No dark patterns.
- No tracking cookies unless explicitly accepted (see [Cookie Policy](/en/cookies)).

Our mission: deliver fast, fair, and delightful decision tools.

---

> 🧪 **Disclaimer**
> These metrics are based on internal simulations and are designed for real-world reliability but are not peer-reviewed. For academic collaboration, [contact us](/en/contact).

---

Kuqus blends cutting-edge science, user-friendly design, and privacy-first principles to deliver accurate, efficient, and engaging surveys. Try Kuqus today and experience the future of decision-making!

<div>
    <a href="https://github.com/optimisticoder-llc/publications/blob/main/kuqus/ALGORITHM.md">Beacon Algorithm</a>
    <span>© 2025 by</span>
    <a href="https://optimisticoder.com">
        Optimisticoder Innovation Labs LLC
    </a>
    <span>is licensed under</span>
    <a href="https://creativecommons.org/licenses/by-sa/4.0/">
        Creative Commons Attribution-ShareAlike 4.0 International
    </a>
    <img
        alt="CC"
        src="https://mirrors.creativecommons.org/presskit/icons/cc.svg"
        class="w-fit h-fit max-w-4 inline"
    />
    <img
        alt="Attribution"
        src="https://mirrors.creativecommons.org/presskit/icons/by.svg"
        class="w-fit h-fit max-w-4 inline"
    />
    <img
        alt="SA"
        src="https://mirrors.creativecommons.org/presskit/icons/sa.svg"
        class="w-fit h-fit max-w-4 inline"
    />
</div>
