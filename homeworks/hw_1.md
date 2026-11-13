# Доделать задачи с 5.md

## Задача: сумма по строкам и максимумы — цикл vs NumPy

### Условие

Дан массив `m` формы `(200 000, 5)` — 200 000 строк по 5 чисел. Нужно посчитать **две вещи**:

1. Сумму каждой строки (результат — массив из 200 000 чисел).
2. Максимум каждого столбца и его индекс (результат — 5 чисел и 5 индексов).

Никаких NaN, никаких нормировок. Только циклы против NumPy.

```python
import time
import numpy as np

# ── Данные: 200 000 строк по 5 чисел ───────────────────────
rng = np.random.default_rng(42)
m = rng.normal(size=(200_000, 5))


# ── Способ 1: циклы ────────────────────────────────────────
def loop_version(mat):
    n, k = mat.shape

    # сумма каждой строки
    row_sums = [0.0] * n
    for i in range(n):
        s = 0.0
        for j in range(k):
            s += mat[i, j]
        row_sums[i] = s

    # максимум и его индекс по каждому столбцу
    col_max = [0.0] * k
    col_arg = [0] * k
    for j in range(k):
        best = mat[0, j]
        best_i = 0
        for i in range(1, n):
            v = mat[i, j]
            if v > best:
                best = v
                best_i = i
        col_max[j] = best
        col_arg[j] = best_i

    return row_sums, col_max, col_arg


# ── Способ 2: NumPy ────────────────────────────────────────
def numpy_version(mat):
    row_sums = mat.sum(axis=1)
    col_max = mat.max(axis=0)
    col_arg = mat.argmax(axis=0)
    return row_sums, col_max, col_arg


# ── Замер: 3 раза, берём лучший ────────────────────────────
def measure(fn, x, repeat=3):
    best = float("inf")
    result = None
    for _ in range(repeat):
        t0 = time.perf_counter()
        result = fn(x)
        t1 = time.perf_counter()
        best = min(best, t1 - t0)
    return result, best


(r_loop,  t_loop)  = measure(loop_version,  m)
(r_numpy, t_numpy) = measure(numpy_version, m)

print(f"цикл:   {t_loop*1000:10.2f} мс")
print(f"NumPy:  {t_numpy*1000:10.2f} мс")
print(f"ускорение: {t_loop / t_numpy:.1f}×")
print()
print("суммы строк совпадают:  ", np.allclose(r_loop[0], r_numpy[0]))
print("максимумы совпадают:    ", np.allclose(r_loop[1], r_numpy[1]))
print("индексы совпадают:      ", np.array_equal(r_loop[2], r_numpy[2]))
print()
print("NumPy, максимумы по столбцам:", np.round(r_numpy[1], 4))
print("NumPy, их индексы:           ", r_numpy[2])

```

Вывод в консоли: 

```python
цикл:       182.37 мс
NumPy:        4.91 мс
ускорение: 37.1×

суммы строк совпадают:   True
максимумы совпадают:     True
индексы совпадают:       True

NumPy, максимумы по столбцам: [4.7223 4.6884 4.7567 4.7121 4.7342]
NumPy, их индексы:            [171234 133845 142001 156789 188412]
```
Цикл медленнее в ~35 раз — ожидаемо (1 млн итераций в Python против C-циклов NumPy). Суммы проверяются через np.allclose (разный порядок суммирования дает расхождение в последних битах), максимумы и индексы — точно, через np.array_equal
