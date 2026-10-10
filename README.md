# heap interview questions: the 9 problems that actually get asked, plus how to answer heap questions out loud

Search "heap interview questions" and you'll mostly get two kinds of results: university lecture PDFs about percolate-up and percolate-down, and GitHub gists listing 18 problems with no explanation of which ones matter. Neither is what you need three weeks before a loop at Amazon.

Here's the thing about heaps: interviewers almost never ask you to define one. They ask you to solve a problem where a heap is the clean answer, and then they probe whether you understand *why* you reached for it. So this breaks into two skills — recognizing the pattern, and explaining your choices in a way that survives follow-up questions.

## Most heap questions are the same pattern wearing different clothes

If a problem involves repeatedly grabbing the smallest or largest thing, or keeping "the top k" of a stream you can't fit in memory, you're in heap territory. The tell-tale phrases:

- "kth largest", "k closest", "top k frequent"
- "merge k sorted…" anything
- "maintain the median as data arrives"
- "process this by priority"
- "the smallest/largest so far"

The pattern that covers the majority of these is simple and worth memorizing as a template: keep a heap capped at size k, and whenever it grows past k, throw out the worst candidate.

python
import heapq

# "best" = largest value here; we keep the k largest seen so far
def keep_top_k(values, k):
    heap = []                      # min-heap of size <= k
    for v in values:
        heapq.heappush(heap, v)
        if len(heap) > k:
            heapq.heappop(heap)    # evicts the smallest, i.e. the worst of the top k
    return heap                    # heap[0] is the kth largest overall


Two details interviewers listen for:

- Use a **min**-heap to keep the k **largest** elements. The root holds the weakest survivor, which is exactly the one you want to compare against and possibly evict.
- Python's `heapq` only does min-heaps. For a max-heap, push negated values. In Java, `PriorityQueue` defaults to min and you pass a comparator to flip it; in C++, `priority_queue<int, vector<int>, greater<int>>` gives you the min-heap.

## The verbal heap questions, and the answers that land

These come up as a warm-up before the coding starts, or as a follow-up while your solution is on screen. Short, specific answers beat lectures.

**What is a heap?** A complete binary tree that satisfies the heap property: every parent is ≤ its children (min-heap) or ≥ them (max-heap). Because it's complete, it fits in a flat array with no pointers — parent of index `i` is `(i - 1) // 2`, left child is `2i + 1`, right child is `2i + 2`.

**What's the name collision?** A heap is a data structure; a priority queue is the abstract type it usually implements. A binary heap is the most common implementation, not the only one (Fibonacci heaps exist and have better amortized decrease-key behavior, which matters in theory and rarely in an interview).

**Complexities worth having instant:**

| Operation | Time |
| --- | --- |
| peek (find min/max) | O(1) |
| insert | O(log n) |
| extract min/max | O(log n) |
| build a heap from n unsorted items | O(n) |
| search for an arbitrary value | O(n) |

That last row is the one people forget, and it's often the answer to "why not use a heap here?"

**Why is building a heap O(n) instead of O(n log n)?** Because you heapify bottom-up, starting from the last internal node. Most nodes sit near the bottom and sift down through almost nothing. If you sum the work by level — the root can fall log n levels, but there are only n/4 nodes at the second-to-last level that can fall one level — the series converges to O(n). Pushing n items one at a time really is O(n log n); that's a different operation.

**Heap vs. BST vs. sorted array?** Sorted array: O(log n) binary search for a value, O(n) insert. BST (balanced): O(log n) for both, and you can retrieve any order. Heap: O(1) for the extreme only, O(log n) insert/extract, and no efficient way to do anything else. Heaps win when you only care about the extreme and you want the cheapest possible implementation.

**When would you *not* use a heap?** When you need the full sorted order (just sort), when you need range queries or predecessor lookups (balanced tree), when you need fast arbitrary search (hash map), or when n is tiny and sorting is simpler to write correctly.

## The 9 problems worth practicing

This list is ordered roughly by how often they appear, and every one maps onto the size-k template above or a close variant.

**1. Kth Largest Element in an Array.** Min-heap of size k, push each element, pop when the size exceeds k. At the end the root is the answer. O(n log k) time, O(k) space. Quickselect gets you O(n) average time if the interviewer pushes on complexity, but it has a bad worst case and most people write it with an off-by-one bug under pressure.

**2. Top K Frequent Elements.** Count with a hash map, then heap the (frequency, value) pairs with the same size-k cap. O(n log k). Mention the bucket-sort alternative that reaches O(n) by using frequency as an array index, since frequency is bounded by n — interviewers tend to like hearing that you know the tradeoff even if you don't code it.

**3. K Closest Points to Origin.** Same template, opposite polarity: you keep the k *smallest* distances, so you use a max-heap of size k and evict the farthest. This is where candidates who memorized "min-heap for k largest" without understanding the invariant get stuck.

**4. Merge k Sorted Lists.** Push the head of each list into a min-heap, pop the smallest, then push the next node from that same list. Total O(N log k) where N is the total number of nodes. The reason it's log k rather than log N is that the heap never holds more than one node per list.

**5. Kth Smallest Element in a Sorted Matrix.** Rows are sorted, which means each row's frontier is a candidate. Heap holds at most one element per row, pop and push the right neighbor k times: O(k log n).

**6. Reorganize String / Rearrange So No Two Adjacent Chars Match.** Max-heap by remaining count, plus a "cooldown" slot holding the character you just used. Greedy and heap-shaped — you always want the currently most frequent character.

**7. Task Scheduler.** Same idea: max-heap of remaining task counts, process the n most frequent tasks per cycle, refill with a cooldown. The heap is doing the "who's most urgent right now" work.

**8. Minimum Cost to Connect Ropes (or Huffman-style merging).** Always merge the two cheapest items, which means a min-heap, and push the merged result back. It's the canonical "greedy plus priority queue" problem.

**9. Find Median from Data Stream.** Two heaps, described below.

Dijkstra's algorithm also belongs on the list in spirit — the priority queue is the whole reason it's O(E log V) instead of something worse — but it usually shows up inside graph rounds rather than as a "heap problem."

## Two heaps: the median problem and its relatives

`Find Median from Data Stream` deserves its own note because the trick isn't the size-k pattern at all. You keep the lower half of the data in a max-heap and the upper half in a min-heap, and you rebalance after every insert so their sizes differ by at most one.

- Insert: O(log n) — one push, maybe one transfer between heaps.
- Find median: O(1) — the root of the larger heap, or the average of both roots if they're equal.

The two-heap idea also shows up in sliding-window median and in problems about splitting a stream into halves by some criterion. The failure mode here is rebalancing lazily or forgetting to check which heap is bigger before returning the median.

## How to answer out loud before you write a line of code

Interviewers score the reasoning as much as the code. A four-sentence opener does most of the work:

> "We need repeated access to the smallest or largest element, so a heap gives O(1) peeks and O(log n) insert and remove. Since we only care about the top k, I'll cap the heap at size k and evict the worst candidate each time the size exceeds k, which keeps time at O(n log k) and space at O(k). Sorting instead would cost O(n log n) and give us more than we need. For k much smaller than n, the heap is strictly better."

That answers *what*, *why*, and *why not the obvious alternative* in about fifteen seconds, and it invites the follow-up you want ("what if k equals n?") instead of the one you don't ("so why is a heap better than sorting?").

## The traps that cost people offers

- **Forgetting build-heap is O(n).** If you claim building a heap is O(n log n), a detail-oriented interviewer will ask you to justify it, and "because inserting n elements is n log n" is wrong when a bulk heapify is available.
- **Getting the polarity backwards.** Min-heap for k largest, max-heap for k smallest. Derive it from the eviction rule instead of memorizing it: you evict the root, so the root must be the candidate you'd most happily lose.
- **Ignoring ties.** "Kth largest" and "top k frequent" both get ambiguous when values or frequencies repeat. State your tie-breaking assumption before coding.
- **Treating `heapq` as a max-heap.** Negate on the way in, negate on the way out, and say that you're doing it. Don't silently produce inverted results.
- **Ignoring the streaming constraint.** "What if the input doesn't fit in memory?" is the classic follow-up, and the size-k heap is usually the answer because it stores k items, not n.
- **Mixing up log k and log n.** The heap size, not the input size, determines the log factor. Saying this unprompted is a small credibility win.

## Practicing this without grinding 3,000 random problems

The reason heap questions feel slippery isn't the code — it's pattern selection under time pressure. You read a problem, spend ten minutes on a sorting approach that nearly works, and run out of clock. That's the specific gap a pattern-first course is built to close, which is where [👉 AlgoMonster](https://algo.monster?fpr=li69) fits into this list.

AlgoMonster is a text-and-diagram course built by engineers who worked at Google, with content organized around coding patterns rather than a raw problem bank. Heap material lands mainly under its top-k and two-heap patterns, alongside decision flowcharts that walk you through questions like "are you repeatedly looking for an extreme value?" before you've committed to an approach. Every pattern comes with a template you adapt, then a handful of problems with small twists — the format the interview actually tests.

A few things worth knowing before you pay:

- There's a **free tier** with limited access to a subset of patterns and problems. That's enough to gauge whether the reading-and-diagrams format suits you, and it's not enough to prep seriously.
- Format is **text and interactive diagrams, no videos**. If you learn by watching someone narrate a whiteboard, this will feel dry.
- The problem count (roughly 231 by most independent counts) is far smaller than LeetCode's 3,000+. The pitch is fewer, better-mapped problems, not volume. Most candidates who succeed pair it with LeetCode for reps.
- **System design and behavioral coverage is basic.** For L5+ loops you need a dedicated resource for those rounds.

### Current plans and pricing

Pricing on AlgoMonster moves with promotions, so treat the numbers below as what the official subscribe page showed at the time of writing. Third-party review sites quote higher list prices (around $198/year and $378 lifetime), which is consistent with the current promotional discounts rather than a contradiction.

| Plan | What you get | Price | Billing | Link |
| --- | --- | --- | --- | --- |
| Free | Limited access to a subset of patterns and problems | $0 | — | Start with the free tier |
| Pro – Yearly | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; 500+ lessons across 6 courses; 1,000+ illustrations; 3,000+ editorials; company questions updated monthly; member discount on live bootcamps; 12 months of access | $5.84/mo (billed $69.99/year) | Annual, cancel anytime | Get Pro Yearly |
| Pro – Lifetime | Everything in the annual plan, plus lifetime updates | $89 | One-time | Get Pro Lifetime |

If your loop is within the next two or three months, the yearly plan is the sensible pick. If you expect to interview again in a couple of years — most engineers do — the lifetime plan is roughly one year's price and stops being a decision you repeat. Both come out cheaper than a single year of LeetCode Premium at its list price, though they're selling different things: patterns versus volume.

## Quick answers to the questions people ask next

**Are heap questions common at big tech companies?** They're a standard mid-level topic rather than a rare one. Top-k and kth-largest variants appear across Google, Amazon, Meta and Apple loops, and priority queues are unavoidable in graph rounds because Dijkstra uses one.

**Do I need to implement a heap from scratch?** Usually no — calling `heapq` or `PriorityQueue` is fine. But you should be able to explain the array indexing and the sift-up/sift-down mechanics if asked, and "implement a min-heap" occasionally shows up as the problem itself.

**How much heap practice is enough?** If you can solve the nine problems above and explain the complexity of each without notes, you're better prepared than most candidates. Do them twice, a week apart, and time the second pass.

**Is a heap ever the wrong answer in an interview?** Yes, and saying so is a plus. If the problem needs fully sorted output, range queries, or repeated lookups of arbitrary keys, a heap is the wrong tool and pointing that out early saves you from writing a solution you'll have to throw away.

The pattern is small, the vocabulary is small, and the number of problems interviewers actually draw from is smaller than the internet makes it look. Learn the size-k template cold, learn the two-heap variant, and the rest is presentation.
