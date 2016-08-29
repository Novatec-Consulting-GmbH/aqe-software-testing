# Tennis Score Kata

## Introduction

Tennis score is a short and fun kata. In a dojo, it can quickly lead to some
discussion about design (e.g. internal representation of the game state).

The original kata is from [codingdojo.org][1]. See [Wikipedia][2] for more
details on tennis rules.

[1]: http://www.codingdojo.org/cgi-bin/index.pl?KataTennis
[2]: http://en.wikipedia.org/wiki/Tennis#Scoring


## Kata

Your task is to implement a display for tennis scores. It will be used during
a live television broadcast of the next championship. No pressure!

Scoring rules in tennis are as follows:

1. Players can have either of these points during a game: 0 (or “Love”), 15,
   30, 40. Players cannot have more than 40 points.
2. If a player has 40 points and scores while her opponent has less than 40,
   she wins the game.
3. If both players have 40 points, the score is "deuce".
4. If the score is "deuce" and a player scores, that player holds "advantage".
5. If a player holds advantage and scores again, she wins the game.
6. If a player holds advantage and loses the ball, she loses the advantage and
   the game score returns to "deuce".

Your score display should track the game and show the names of the players,
their points in the current game and indicate which player holds advantage.

Suggestion for an initial interface, just to get started:

```java
@Test public void
displayScoreWithFifteenForPlayerA() {
    ScoreBoard scoreBoard = new ScoreBoard("Nadal", "Federer");
    scoreBoard.scorePointPlayerA();
    assertThat(scoreBoard.getDisplayedScore(), is("Nadal 15 : 0 Federer"));
}
```

## Challenges

* Extend your implementation to track and give an overview of the currently
  played set. A set is a sequence of games. Typically a player wins a set when
  she has won at least two games more than her opponent after at least six
  games have been played.
* Extend your implementation even more, so that one can track an entire match
  using your display software. A match is a sequence of sets and is usually
  decided through either a best of three or a best of five sets system.

