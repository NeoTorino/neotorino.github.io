# Shells



| Name | Path | FreeBSD |
|---|---|---|
| Bourne shell  | /bin/sh | Installed |
| Cshell | /bin/csh | Installed |
| TENEX C shell | /bin/tcsh | Installed |
| Bourne-again shell  | /path/to/bin/bash | Available |
| Z shell | /path/to/bin/zsh | Available |
| Korn shell | /path/to/bin/kcsh | Available |

## Print 1 to 5" script

### Bourne shell (sh)
```
#!/bin/sh

echo "Bourne shell loop:"

i=1
while [ $i -le 5 ]
do
  echo "Number: $i"
  i=`expr $i + 1`
done
```
This uses traditional Bourne syntax: while, test [ ], expr for arithmetic.


### Cshell (csh)
```
#!/bin/csh

echo "C shell loop:"

set i = 1
while ( $i <= 5 )
  echo "Number: $i"
  @ i = $i + 1
end
```
In csh, variables use set, and arithmetic uses @. Also, conditions are in parentheses () instead of brackets [].


### TENEX C shell
```
#!/bin/tcsh

echo "TENEX C shell loop:"

set i = 1
while ( $i <= 5 )
    echo "Number: $i"
    @ i = $i + 1
end
```
This is almost identical to csh, because tcsh is a compatible superset of csh.
However, tcsh allows you to use nicer editing, filename completion, and better scripting practices if needed.

```
#!/bin/tcsh

echo "TENEX C shell foreach loop:"

foreach i (1 2 3 4 5)
    echo "Number: $i"
end
```
A slightly more tcsh-style version (using foreach, which is a little cleaner)

