# Tic Tac Toe project notes

## The basics

yada yada yada

## Board

The board class' `cells` attribute is an array of nine spaces. I would prefer `nil` as opposed to `" "`, because the logic is not visual, so why use whitespace? If empty spaces were `nil` you could test for presence with just their identity.

## Artificial Intelligence

### Methods

There are only three real positions you can place on the first turn.

```
 1 | 2 |
-----------
   | 3 |
-----------
   |   |
```

The board can be turned to match one of those positions. Try modeling it as a turnable map, might make your algorithm simpler.