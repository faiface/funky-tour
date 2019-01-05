# Putting it together: a calculator

```funky
func main : IO =
    print "Type X op Y: ";
    scan \x-str
    scan \op
    scan \y-str
    let (float x-str) \x
    let (float y-str) \y
    println (
        if (op == "+") (string (x + y));
        if (op == "-") (string (x - y));
        if (op == "*") (string (x * y));
        if (op == "/") (string (x / y));
        "invalid operator: " ++ op
    );
    quit
```
