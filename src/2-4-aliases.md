# Aliases

Aliases are sooo simple! Too simple. All they do is they define **a new name for an existing type**:

```funky
alias Name = type
```

That's it!

The `String` type is an alias:

```funky
alias String = List Char
```

Therefore, `String` and `List Char` are perfect synonyms.

Of course, aliases **can also contain type variables** and have one interesting, although not very useful feature: **they can be recursive**. I haven't found a situation yet where a recursive alias would be needed, but I'll leave that up to you. ;)
