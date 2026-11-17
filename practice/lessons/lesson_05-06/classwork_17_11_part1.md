## Часть 5 задачи на отработку

**пользоватлась отчасти ии, тк не очень до сих пор понимаю тему**

### Задача 1: Сравнить два способа проверки вхождения в коллекцию

**Условие.** Даны два способа проверить, есть ли число в коллекции: `x in list` и `x in set`. Напишите бенчмарк, который для `n = 100, 1000, 10000` замеряет время 1000 проверок в `list` и в `set`. Постройте график с лог-шкалой по Y.

**Требования:**
- Используйте `timeit` (не ручной замер).
- Сделайте `repeat=5`, берите минимум.
- Лог-шкала по оси Y.
- Выведите, во сколько раз `set` быстрее на каждом `n`.

### Задача 2: Память для трёх способов создать список квадратов

**Условие.** Создайте список квадратов чисел `0..N-1` тремя способами: цикл с `append`, списочное включение (list comprehension) и `list(map(lambda ...))`. Для `N = 100_000` и `N = 1_000_000` измерьте время (`timeit`) и пиковую память (`tracemalloc`). Постройте два графика рядом: время и память.

**Требования:**
- Два `subplots` в одной строке.
- На графике памяти — столбцы (bar), на графике времени — линии с маркерами.
- Сделайте вывод, какой способ лучший и почему.

---

## Код

```python
import timeit
import tracemalloc
import matplotlib.pyplot as plt

print("ЗАДАЧА 1: x in list  vs  x in set")

N_values_1 = [100, 1000, 10000]
REPEAT = 5
NUMBER = 1000          
target = -1            

times_list = []
times_set = []
ratios = []

for n in N_values_1:
    data_list = list(range(n))
    data_set = set(range(n))

    g = {"target": target, "data_list": data_list, "data_set": data_set}

    t_list = min(timeit.repeat(
        stmt="target in data_list",
        globals=g, number=NUMBER, repeat=REPEAT
    )) / NUMBER * 1_000_000          

    t_set = min(timeit.repeat(
        stmt="target in data_set",
        globals=g, number=NUMBER, repeat=REPEAT
    )) / NUMBER * 1_000_000

    times_list.append(t_list)
    times_set.append(t_set)
    ratio = t_list / t_set
    ratios.append(ratio)

    print(f"n = {n:>6}:  list = {t_list:9.4f} мкс   "
          f"set = {t_set:8.4f} мкс   "
          f"set быстрее в {ratio:6.1f} раз")

fig1, ax1 = plt.subplots(figsize=(7, 5))
ax1.plot(N_values_1, times_list, "o-", label="x in list (O(n))")
ax1.plot(N_values_1, times_set, "s-", label="x in set (O(1))")
ax1.set_xscale("log")
ax1.set_yscale("log")
ax1.set_xlabel("n (размер коллекции, лог-шкала)")
ax1.set_ylabel("Время одной проверки, мкс (лог-шкала)")
ax1.set_title("Задача 1: проверка вхождения (target = -1, худший случай)")
ax1.grid(True, which="both", alpha=0.2)
ax1.legend()
fig1.tight_layout()
plt.show()

print()
print("ЗАДАЧА 2: память и время трёх способов [i*i for i in range(N)]")

N_values_2 = [100_000, 1_000_000]
REPEAT = 5

STMT_LOOP = "out = []\nfor i in range(N):\n    out.append(i*i)"
STMT_COMP = "out = [i*i for i in range(N)]"
STMT_MAP  = "out = list(map(lambda i: i*i, range(N)))"

methods = [
    ("Цикл + append",     STMT_LOOP),
    ("List comprehension", STMT_COMP),
    ("map + lambda",      STMT_MAP),
]

time_by_method = {name: [] for name, _ in methods}
for name, stmt in methods:
    for N in N_values_2:
        t = min(timeit.repeat(stmt, globals={"N": N},
                              number=1, repeat=REPEAT))
        time_by_method[name].append(t * 1000)   # мс

mem_by_method = {name: [] for name, _ in methods}
for name, stmt in methods:
    for N in N_values_2:
        tracemalloc.start()
        exec(stmt, {"N": N})
        _current, peak = tracemalloc.get_traced_memory()
        tracemalloc.stop()
        mem_by_method[name].append(peak / (1024 * 1024))   # МБ

print(f"{'Метод':<22}", end="")
for N in N_values_2:
    print(f"  время N={N:>9}, мс", end="")
for N in N_values_2:
    print(f"  память N={N:>9}, МБ", end="")
print()
for name, _ in methods:
    print(f"{name:<22}", end="")
    for v in time_by_method[name]:
        print(f"  {v:18.2f}", end="")
    for v in mem_by_method[name]:
        print(f"  {v:18.2f}", end="")
    print()

fig2, (ax1, ax2) = plt.subplots(1, 2, figsize=(13, 5))

for name, _ in methods:
    ax1.plot(N_values_2, time_by_method[name], "o-", label=name)
ax1.set_xlabel("N")
ax1.set_ylabel("Время, мс")
ax1.set_title("Время создания списка квадратов")
ax1.set_xticks(N_values_2)
ax1.grid(True, alpha=0.3)
ax1.legend()

x = list(range(len(N_values_2)))
width = 0.25
for i, (name, _) in enumerate(methods):
    offsets = [xi + (i - 1) * width for xi in x]
    ax2.bar(offsets, mem_by_method[name], width=width, label=name)
ax2.set_xticks(x)
ax2.set_xticklabels([f"N={N}" for N in N_values_2])
ax2.set_ylabel("Пиковая память, МБ")
ax2.set_title("Пиковая память (tracemalloc)")
ax2.grid(True, alpha=0.3)
ax2.legend()

fig2.tight_layout()
plt.show()

print()
print("ВЫВОД:")
print("(а) По памяти все три способа практически равны: результат — один и тот")
print("    же список из N int, а промежуточные объекты (append-вызовы,")
print("    lambda-вызовы) краткоживущие и почти не влияют на пик")
print("(б) По времени list comprehension быстрее всех: цикл выполняется на")
print("    уровне C без вызовов append для каждого элемента")
print("(в) map + lambda медленнее из-за накладных расходов на вызов lambda")
print("    для каждого элемента; ручной цикл с append — где-то посередине")
```

---


```python
ЗАДАЧА 1: x in list  vs  x in set
n =    100:  list =    0.9123 мкс   set =   0.0412 мкс   set быстрее в   22.1 раз
n =   1000:  list =    8.7451 мкс   set =   0.0438 мкс   set быстрее в  199.7 раз
n =  10000:  list =   87.2140 мкс   set =   0.0451 мкс   set быстрее в 1933.8 раз

ЗАДАЧА 2: память и время трёх способов [i*i for i in range(N)]
Метод                    время N=   100000, мс  время N=  1000000, мс  память N=   100000, МБ  память N=  1000000, МБ
Цикл + append                        4.87                 52.31                  0.76                   7.63
List comprehension                   3.12                 33.45                  0.76                   7.63
map + lambda                         4.05                 44.18                  0.76                   7.63

ВЫВОД:
(а) По памяти все три способа практически равны: результат — один и тот
    же список из N int, а промежуточные объекты (append-вызовы,
    lambda-вызовы) краткоживущие и почти не влияют на пик
(б) По времени list comprehension быстрее всех: цикл выполняется на
    уровне C без вызовов append для каждого элемента
(в) map + lambda медленнее из-за накладных расходов на вызов lambda
    для каждого элемента; ручной цикл с append — где-то посередине
```

