# Generaliseable Unique Ordered Pairing Function

This article presents a function for encoding unique ordered sets of natural numbers into a single unique natural number.

When encoding all combinations of unique ordered sets up to M (where M is the largest set) the resulting numbers will range from 0 to N-1 for all N combinations. This means the function can be used as a minimal perfect hash function in that situation.

Finally this function can be easily generalised to higher dimensions.

```
(0, 1) = 0
(0, 2) = 1
(1, 2) = 2
(0, 3) = 3
(1, 3) = 4
(2, 3) = 5

(0, 1, 2) = 0
(0, 1, 3) = 1
(0, 2, 3) = 2
(1, 2, 3) = 3
```

## Why?

I needed a way to index component archetypes in an ECS I was making. While pairing functions such as Cantor pairing function or Szudzik's elegant pair could be used, because each archetype could only consist of a unique combination of components it meant there would be a lot of waste (the indexes would be sparse).

## How it works

### In 2D

First let's start with sets of two. If we mapped out every valid combination of these unique ordered sets up to 5 (from {0,1} to {4,5}) it would give us the following grid (where x is a valid combination):

```
      0     1     2     3     4    
   + ——— + ——— + ——— + ——— + ——— +
 0 |     |     |     |     |     |
   + ——— + ——— + ——— + ——— + ——— +
 1 |  x  |     |     |     |     |
   + ——— + ——— + ——— + ——— + ——— +
 2 |  x  |  x  |     |     |     |
   + ——— + ——— + ——— + ——— + ——— + y-axis
 3 |  x  |  x  |  x  |     |     |
   + ——— + ——— + ——— + ——— + ——— +
 4 |  x  |  x  |  x  |  x  |     |
   + ——— + —-— + ——— + ——— + ——— +
 5 |  x  |  x  |  x  |  x  |  x  |
   + ——— + ——— + ——— + ——— + ——— +
                x-axis
```

From this we can see the shape these 15 unique combinations produce is a right triangle. So we could say the number of unique combinations is the area of this triangle (5*6/2 = 15).

Knowing this we could encode each set as the area of the right triangle with y height and y - 1 length (which will give us a result that's larger than any smaller set), plus x. Or `((y - 1) * y / 2) + x`. Doing that would give us the resulting mapped combinations:

```
      0     1     2     3     4   
   + ——— + ——— + ——— + ——— + ——— +
 1 |  0  |     |     |     |     |
   + ——— + ——— + ——— + ——— + ——— +
 2 |  1  |  2  |     |     |     |
   + ——— + ——— + ——— + ——— + ——— +
 3 |  3  |  4  |  5  |     |     |
   + ——— + ——— + ——— + ——— + ——— +
 4 |  6  |  7  |  8  |  9  |     |
   + ——— + ——— + ——— + ——— + ——— +
 5 | 10  | 11  | 12  | 13  | 14  |
   + ——— + ——— + ——— + ——— + ——— +
```

This can then keep going to infinity.

### In 3D

Continuing from this we can scale this up to sets of three by moving the problem to 3 dimensions and accounting for the volume of the trirectangular tetrahedron (of z, z-1, z-2 lengths) to give us a result that's larger than any smaller set. Then like previously we can add the result of the 2D encoding to give us the final encoding. Or `((z - 2) * (z - 1) * z / 6) + ((y - 1) * y / 2) + x`. Doing that would give us the resulting mapped combinations (planes separated by their z-axis, starting at 2)

```
           z=2                      z=3                    z=4
      0     1     2            0     1     2          0     1     2              
   + ——— + ——— + ——— +      + ——— + ——— + ——— +    + ——— + ——— + ——— +
 1 |  0  |     |     |      |  1  |     |     |    |  4  |     |     |
   + ——— + ——— + ——— +      + ——— + ——— + ——— +    + ——— + ——— + ——— +
 2 |     |     |     |      |  2  |  3  |     |    |  5  |  6  |     | y-axis
   + ——— + ——— + ——— +      + ——— + ——— + ——— +    + ——— + ——— + ——— +
 3 |     |     |     |      |     |     |     |    |  7  |  8  |  9  |
   + ——— + ——— + ——— +      + ——— + ——— + ——— +    + ——— + ——— + ——— +
         x-axis
```

### Generalising For Higher Dimensions

While we can keep continuing this for higher dimensions, we should probably now generalise it.

For each element/axis (x) in the set at index/dimension (n) we do: ![\frac{\prod_{i=0}^{n}(x-{i})}{(n+1)!}](https://latex.codecogs.com/svg.latex?\Large&space;%5Cfrac%7B%5Cprod_%7Bi%3D0%7D%5E%7Bn%7D%28x-%7Bi%7D%29%7D%7B%28n+1%29%21%7D)

And sum the result for each dimension.

## Examples

Note: In the following examples we avoid calculating the factorial independently.

```c
// C
unsigned int encode(unsigned int *set, size_t n)
{
    unsigned int result = 0;
    unsigned int factorial = 1;

    for (size_t i = 0; i < n; i++)
    {
        factorial *= (i + 1) * 1;

        unsigned int p = 1;
        for (size_t j = 0; j <= i; j++)
        {
            p *= (set[i] - j);
        }

        result += p / factorial;
    }

    return result;
}

encode((unsigned int[]){ 0, 4, 6 }, 3); //=> 26
encode((unsigned int[]){ 0, 1, 2, 3, 4, 5, 7, 9 }, 8); //=> 10
```

```elixir
# Elixir
defmodule Set do
    def encode(set, n \\ 1, factorial \\ 1)
    def encode([], _, _), do: 0
    def encode([x|set], n, factorial), do: div(product(x, n - 1), (factorial * n)) + encode(set, n + 1, factorial * n)

    defp product(x, 0), do: x
    defp product(x, n), do: (x - n) * product(x, n - 1)
end

Set.encode [0,4,6] #=> 26
Set.encode [0,1,2,3,4,5,7,9] #=> 10
```
