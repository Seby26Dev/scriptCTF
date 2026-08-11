
# This New One 2 

```
Has a very unique wishlist! Can you find what it's hiding?
```

The first step was to realize that John Doe also has a Discord user.
The pivot is realizing the wishlist the challenge is refering is the user's Discord wishlist.

Putting each item's name in a file and taking the first letter, top-to-bottom / left-to-right revealed the flag:


| #  | Item              | Letter |
|----|-------------------|--------|
| 1  | The Hermit        | T      |
| 2  | South Korea       | S      |
| 3  | Enchanted Forest  | E      |
| 4  | Brazil            | B      |
| 5  | Ecuador           | E      |
| 6  | He-Bat            | H      |
| 7  | The Tower         | T      |
| 8  | Oni Mask          | O      |
| 9  | France            | F      |
| 10 | Haiti             | H      |
| 11 | Saudi Arabia      | S      |
| 12 | Iraq              | I      |
| 13 | Woody             | W      |

Forward: TSEBEHTOFHSIW - gibberish. But it's a wishlist, so read it backwards:

W I S H F O T H E B E S T -> "WISH FO THE BEST"

Flag: scriptCTF{WISHFOTHEBEST}
