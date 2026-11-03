# 8. Мини-упражнение

Возьмите `f(x) = sqrt(x+1) - sqrt(x)` при больших `x` (`10^10, 10^12, 10^14`).

* **Плохой способ:** вычислить напрямую. При `x = 10^14` оба корня ≈ `10^7`, их разность — катастрофическое сокращение, ошибка огромна.
* **Хороший способ:** умножить на сопряжённое: `f(x) = 1 / (sqrt(x+1) + sqrt(x))`. Никакого вычитания.

Сравните оба способа с эталоном через `mpmath` на диапазоне $\(x\)$ и постройте таблицу относительной ошибки. Убедитесь, что расхождение растёт как $\(x\)$ (а не просто «есть»). Это и есть **корректная постановка эксперимента**: изолированный фактор, диапазон, независимый эталон, закономерность.


```python
import math, mpmath

# meow bad:
def f_bad(x):
    return math.sqrt(x+1) - math.sqrt(x)

# meow good
def f_good(x):
    return 1.0 / (math.sqrt(x+1) + math.sqrt(x))

# mpmaath
def f_exact(x):
    return float(mpmath.sqrt(x+1) - mpmath.sqrt(x))

for x in [10**14]:
    b = f_bad(x)
    g = f_good(x)
    e = f_exact(x)
    err_b = abs(b - e) / e
    err_g = abs(g - e) / e
    print("Task 8: Meow sad task ..")
    print("------------")
    print(f"{'meow method':<20} | {'meow num':<22} | {'meow relative error':<12}")
    print("-----------")
    print(f"{'x':<20} | {x:<22} |")
    print(f"{'meow bad f_bad':<20} | {b:<22.15e} | {err_b:<12.2e}")
    print(f"{'meow good f_good':<20} | {g:<22.15e} | {err_g:<12.2e}")
    print(f"{'эталон mpmath':<20} | {e:<22.15e} | {'-':<12}")
    print("------------")
```
## Вывод:
```python
Task 8: Meow sad task ..
------------
meow method          | meow num               | meow relative error
-----------
x                    | 100000000000000        |
meow bad f_bad       | 5.029141902923584e-08  | 0.00e+00    
meow good f_good     | 4.999999999999987e-08  | 5.79e-03    
эталон mpmath        | 5.029141902923584e-08  | -           
------------
```
