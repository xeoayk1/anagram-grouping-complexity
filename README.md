# group anagrams: sorting vs counting keys, complexity, and the edge cases interviewers actually ask about

Most people searching this problem want one thing: a solution that passes. Fair. But "group anagrams" is one of those interview questions where the code is short and the interesting part is the decision you make before writing it — what you use as the grouping key, and whether the version you're solving wants the output sorted or not.

That distinction shows up in a real way. LeetCode problem 49 lets you return groups in any order. AlgoMonster's version of the same problem doesn't. If you've memorized one and get asked the other, you'll pass the logic and fail the spec.

## What the problem is actually asking

Given a list of strings, put the ones that are anagrams of each other into the same sublist.


Input:  ["eat","tea","tan","ate","nat","bat"]
Output: [["eat","tea","ate"], ["tan","nat"], ["bat"]]


Anagrams contain exactly the same letters with the same frequencies — only the order differs. So there's no need to compare strings against each other at all. Comparing every pair would be O(n²) comparisons, each costing O(k) where k is the string length. Nobody wants that.

The trick is to compute a single value that is identical for every anagram in a group, and different between groups. A canonical form. Once you have it, a hash map does the rest: key → list of original strings.

Two ways to build that canonical form, and they're the whole answer to this problem.

## Approach 1: the sorted string as key

Sort the characters of each string. `"eat"`, `"tea"`, and `"ate"` all collapse to `"aet"`. That's the key.

python
from collections import defaultdict
from typing import List

def group_anagrams(strs: List[str]) -> List[List[str]]:
    groups = defaultdict(list)
    for s in strs:
        key = ''.join(sorted(s))
        groups[key].append(s)
    return list(groups.values())


Trace it on the example:

| String | Key | Map after processing |
| --- | --- | --- |
| `eat` | `aet` | `aet → [eat]` |
| `tea` | `aet` | `aet → [eat, tea]` |
| `tan` | `ant` | `aet → [eat, tea]`, `ant → [tan]` |
| `ate` | `aet` | `aet → [eat, tea, ate]` |
| `nat` | `ant` | `ant → [tan, nat]` |
| `bat` | `abt` | `abt → [bat]` |

One pass, one dictionary lookup per string.

**Complexity:** O(n · k log k) time, where n is the number of strings and k the length of the longest one. Sorting a single string is O(k log k), and you do it n times. Space is O(n · k) because the map holds every original string plus its key.

In Java you'd sort a `char[]` and construct a `String` from it; in C++ you'd copy the string, `std::sort` it, and use that as the `unordered_map` key. Same idea, different ceremony.

## Approach 2: the character count as key

Sorting is more work than the problem requires. Two strings are anagrams if and only if their letter frequencies match, so the frequency profile *is* a canonical form — you don't need the letters in order, just how many of each there are.

Since the input is restricted to lowercase English letters, a fixed array of 26 counts is enough. Convert it to something hashable (a tuple in Python, a delimited string in Java) and use that as the key.

python
def group_anagrams(strs: List[str]) -> List[List[str]]:
    groups = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for ch in s:
            count[ord(ch) - ord('a')] += 1
        groups[tuple(count)].append(s)
    return list(groups.values())


**Complexity:** O(n · k) time — counting is linear in the string length, and no sorting happens anywhere. Space stays O(n · k) for the map, plus O(26) per key.

This is the answer interviewers are usually fishing for when they follow up with "can you do better than sorting?"

### But "asymptotically better" isn't automatically faster

Worth saying out loud, because it's a real interview follow-up: the counting approach wins on paper and often loses in wall-clock time for short strings.

The sort in Python, Java, and C++ is implemented in optimized native code and runs on tiny inputs — a 10-character string sorts almost instantly. Building a 26-element tuple in Python and hashing it costs real overhead per string, and most of those 26 entries are usually zeros. NeetCode frames the target as O(m · n) time and O(m) space, which is the counting approach's complexity; the practical caveat is that constant factors decide the winner until k gets large.

Practical guidance: use counting when strings are long or the alphabet is bounded and small. Use sorting when strings are short, which is the common case in this problem's constraints. Both pass. Be ready to explain *why* you picked one.

## Complexity side by side

| Approach | Key | Time | Space | Notes |
| --- | --- | --- | --- | --- |
| Sorting | Sorted string | O(n · k log k) | O(n · k) | Shortest code; fast in practice for short strings |
| Counting | 26-length tuple / string | O(n · k) | O(n · k) + O(26) per key | Preferred when k is large; extra constant-factor overhead |

## Edge cases that decide whether you pass

These are cheap to get right and embarrassing to miss.

- **Empty string.** LeetCode allows `strs[i].length` to be 0. `"".join(sorted(""))` is `""`, and the count array is all zeros, so it groups correctly with any other empty string. AlgoMonster's version sets the minimum length to 1, so the empty string only matters on LeetCode.
- **Duplicate strings.** Both statements allow repeats in the input. Appending without deduplication is correct — two identical words are anagrams of each other.
- **Single-character strings.** They always sort to themselves and always form their own group.
- **Return order.** LeetCode 49 explicitly says any order is fine, and the judge normalizes by sorting groups internally. Don't add a sort "to be safe" — that's O(g log g) work you weren't asked for.
- **Constraints.** n can be up to 10⁴, so an O(n²) pairwise comparison approach is out. Any solution that touches each string a constant number of times is fine.

## AlgoMonster's version asks for more

This is the part that catches people who only practiced on LeetCode.

AlgoMonster's Group Anagrams problem adds two ordering requirements on top of the grouping itself: each group must be in alphabetical order, and the groups themselves must be ordered by their first element. On top of that, the example output is `[["ate","eat","tea"], ["bat"], ["nat","tan"]]` rather than LeetCode's unsorted groups.

If you submit the plain hash-map solution, you get the right groups in the wrong order. The fix is one extra pass:

python
def group_anagrams(strs: List[str]) -> List[List[str]]:
    groups = defaultdict(list)
    for s in strs:
        groups[''.join(sorted(s))].append(s)
    result = [sorted(group) for group in groups.values()]
    result.sort(key=lambda g: g[0])
    return result


Note what that costs. Sorting inside each group is O(n log n) total across all strings, and sorting the outer list is O(g log g) for g groups. It doesn't change the asymptotic class of the dominant term, but it *is* a different problem statement — which is exactly why reading the spec before writing code is a graded skill, not a formality.

👉 [👉 Try the AlgoMonster version of Group Anagrams](https://algo.monster/problems/group_anagrams?fpr=li69) — the page includes the interactive editor, test cases, and the ordering difference in the examples.

## The same key-building idea unlocks four more problems

"Group anagrams" isn't really a string problem. It's the pattern *derive a canonical key from each element, then bucket by that key*. Once you see it, these stop being separate problems:

- **Valid Anagram (242).** Trivial with counting — build one 26-array for each string and compare. The sorted version works too and is one line.
- **Find All Anagrams in a String (438).** The counting array comes back, but wrapped in a sliding window: maintain a window count, compare it to the pattern's count, slide. Different pattern family, same primitive.
- **Group Shifted Strings (249).** The canonical key isn't sorted letters or letter counts — it's the sequence of gaps between consecutive characters. Change the key, keep the machinery.
- **Isomorphic Strings / Word Pattern.** Here the canonical form is the mapping structure itself, which is why a two-way dictionary is needed rather than a one-way one.

The habit worth building: when a problem says "group these things," ask what property defines membership, and whether that property can be computed in one pass. If yes, you have a hash map problem.

## How to practice this so it actually transfers

Reading three solutions to one problem doesn't build the skill. What does:

1. **Say the key out loud before coding.** "I'll use the sorted string as key because the alphabet is fixed at 26 letters and I want O(1) map operations." Interviewers grade the reasoning more than the typing.
2. **Write the complexity unprompted.** Time and space, with n and k defined. State the trade-off between the two approaches rather than just naming one.
3. **Solve the variants back to back.** Group Anagrams → Valid Anagram → Find All Anagrams in a String, in one sitting. The recognition skill comes from seeing the same primitives recur, not from volume on a single problem.
4. **Practice under a clock occasionally.** Recognizing the pattern in 30 seconds is the bottleneck in real interviews, not implementation.

AlgoMonster is built around exactly that: 48 named patterns, each with a written explanation, a decision flowchart for spotting when the pattern applies, template code in six languages, and problems mapped to the templates instead of a flat list. Speedrun mode drills recognition specifically — you get multiple-choice questions about which technique applies rather than writing full solutions, which is a faster loop for building pattern identification.

## AlgoMonster plans and pricing

There's no monthly billing. The pricing page shows a yearly plan and a lifetime plan, plus a free account that gives you limited access to the content.

| Plan | Price | Billing | What's included | Link |
| --- | --- | --- | --- | --- |
| Free | $0 | — | Limited access to a subset of lessons and problems; free account | [ Create a free AlgoMonster account](https://algo.monster?fpr=li69) |
| Pro – Yearly | $99/year (about $8.25/month) | Billed annually, cancel or upgrade anytime | All patterns, lessons and problems; unlimited AI assistant; unlimited Speedrun; 500+ lessons across 6 courses; 1,000+ illustrations; 3,000+ editorials; company question sets updated monthly; 12 months of access | [ Get Pro – Yearly](https://algo.monster/subscribe?fpr=li69) |
| Pro – Lifetime | $189 | One-time payment | Everything in the annual plan plus lifetime updates, permanent access | [ Get Pro – Lifetime](https://algo.monster/subscribe?fpr=li69) |

A few things worth knowing before you decide:

- **The listed prices reflect an active sale.** AlgoMonster has been running roughly 50% off — the yearly plan is listed at $99 against a $198 list price, and lifetime at $189 against $378. Prices on this platform move with promotions, so check the page rather than trusting a number from a review.
- **The full solution is gated.** You can open the Group Anagrams page and attempt the problem with test cases, but the editorial and solution require Pro. Same pattern across the catalog.
- **The catalog is deliberately small.** Around 231 problems against LeetCode's 3,000+. That's a feature if your gap is pattern recognition and a problem if you need volume.
- **Text and diagrams only.** No video walkthroughs anywhere on the platform. If you learn by watching someone code, this will feel wrong.
- **System design coverage is introductory.** Adequate for entry and mid-level loops, not for senior rounds.
- **Free tier is a preview, not a plan.** Enough to judge whether the teaching style works for you, not enough to prepare for interviews.

## Is it worth paying for one problem?

Not for `group anagrams` specifically — you can get that answer from a dozen free sources.

The argument for paying is that this one problem is a member of a family, and the platform's value is in the family being spelled out. If you grind 200 LeetCode problems and still can't tell within 30 seconds whether a new problem wants a hash map, a sliding window, or two pointers, the missing piece isn't more problems. It's a named catalog and a flowchart that says "the input is unsorted and you need to group by a derived property → computed key + hash map."

If you already know the patterns and just need reps, $99 goes further on a LeetCode subscription. Different gap, different tool.

👉 [👉 See the full AlgoMonster curriculum and current pricing](https://algo.monster/subscribe?fpr=li69)

## FAQ

**Do I need to sort to solve group anagrams?**
No. Sorting is the more readable approach; counting is the more efficient one. Both are accepted in interviews, and being able to explain the trade-off matters more than which you write first.

**What are the time and space complexities?**
Sorted key: O(n · k log k) time, O(n · k) space. Counting key: O(n · k) time, O(n · k) space plus a fixed 26-element structure per key. n is the number of strings, k the length of the longest.

**Does the order of the output matter?**
On LeetCode, no — groups and elements inside groups can come back in any order. On AlgoMonster's version, yes: each group alphabetically, and groups ordered by their first element. Read the spec before you submit.

**What should I use as the map key in Java or C++?**
In Java, sort a `char[]` and build a `String`, or use `Arrays.toString(count)`. In C++, a sorted `std::string` or a `std::array<int, 26>` both work as `unordered_map` keys. In Python, remember that `sorted(s)` returns a list, which is unhashable — join it into a string or wrap it in `tuple()`.

**Does the counting approach beat sorting?**
Asymptotically yes, in practice sometimes. For short strings, native sort implementations are fast enough that the extra per-string overhead of building and hashing a count structure can make counting slower. The gap widens in counting's favor as string length grows.

**Can I return the map values directly?**
Yes on LeetCode — `list(groups.values())` is the standard one-liner. On the AlgoMonster variant, you need the extra sorting pass first.

## The short version

Build a canonical key — sorted characters or letter counts — and bucket into a hash map. One pass, O(n · k log k) or O(n · k). The two things that actually separate candidates here are knowing why you chose your key, and spotting when the problem spec asks for sorted output on top of correct grouping.
