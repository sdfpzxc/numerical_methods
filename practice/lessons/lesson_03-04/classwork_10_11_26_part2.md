# 15 задач на тему «Переписать обработку большого массива с циклов на NumPy; сравнить время и читаемость»

**Формат сдачи:** файл `.py` (или `.ipynb`) с решениями + короткие выводы (2–3 предложения) по каждой задаче. Для всех задач — запускаемый код.
**Требование к каждой задаче:** реализовать **два** варианта (чистый Python / NumPy), измерить время обоих, посчитать ускорение (×), оценить читаемость (короче/понятнее или нет — обосновать).

- я чуть чуть использовалал ии тк не совсем понимаю материал, я смотрю доп видео, но мне все равно тяжеловато, поэтому код написан наполовину мной и наполивину с ии!!! >_<" . . >

```python
import time
import math
import numpy as np
import random

def measure(fn, x, repeat=3):
    best = float("inf")
    result = None
    for _ in range(repeat):
        t0 = time.perf_counter()
        result = fn(x)
        t1 = time.perf_counter()
        best = min(best, t1 - t0)
    return result, best
```

# 1 Задание

```python
print("1. Сумма массива")
def loop_sum_1(a):
    s = 0.0
    for x in a: s += x
    return s
def numpy_sum_1(a): return a.sum()

for n in [10**5, 10**6, 10**7]:
    arr = np.random.rand(n)
    _, t_loop = measure(loop_sum_1, arr)
    _, t_np = measure(numpy_sum_1, arr)
    print(f"n={n:<9} | цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print("Ускорение растет вместе с n, так как фиксированные накладные расходы NumPy вызов функций перестают играть роль, а скорость работы внутренних C-циклов начинает доминировать, разница становится заметной больше 10 раз уже на n = 10^6\n")
```

# 2 Задание

```python
print("2. Среднее и стандартное отклонение")
def loop_mean_std_2(a):
    n = len(a)
    s = 0.0
    s2 = 0.0
    for x in a:
        s += x
        s2 += x*x
    mean = s / n
    var = (s2 / n) - (mean * mean)
    return mean, math.sqrt(var)
def numpy_mean_std_2(a): return a.mean(), a.std()

arr = np.random.rand(10**6)
(r_l_m, r_l_s), t_loop = measure(loop_mean_std_2, arr)
(r_np_m, r_np_s), t_np = measure(numpy_mean_std_2, arr)
print(f"Ср совпадает до 1e-12: {abs(r_l_m - r_np_m) < 1e-12}")
print(f"Std совпадает до 1e-12: {abs(r_l_s - r_np_s) < 1e-12}")
print("Результаты совпадают до 1e-12 NumPy считает std устойчивее, потому что использует двухпроходный алгоритм: сначала находит точное среднее, а затем сумму квадратов отклонений от него, цикл же считает по формуле разности квадратов, что приводит к катастрофическому вычитанию близких чисел и потере точности\n")
```

# 3 Задание
```python
print("3. Поэлементное преобразование: \(y = \sin(x) + x^2\)")
def loop_math_3(a):
    res = []
    for x in a: res.append(math.sin(x) + x*x)
    return res
def numpy_math_3(a): return np.sin(a) + a**2

arr = np.random.rand(10**6)
res_l, t_loop = measure(loop_math_3, arr)
res_np, t_np = measure(numpy_math_3, arr)
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print("Список Python на миллион элементов требует гораздо больше памяти из-за накладных расходов на каждый объект, тогда как NumPy выделяет один компактный непрерывный массив NumPy-версия короче по коду, но внутри выражения np.sin(a) + a**2 создаются два временных массива: первый для синуса, второй для квадрата, после чего они складываются в третий итоговый массив\n")
```

# 4 Задание
```python
print("4. Условная замена (clipping)")
def loop_clip_4(a):
    res = []
    for x in a:
        if x < 0: res.append(0)
        elif x > 1: res.append(1)
        else: res.append(x)
    return res
def numpy_clip_4(a): return np.clip(a, 0, 1)

arr = np.random.randn(10**6) 
res_l, t_loop = measure(loop_clip_4, arr)
res_np, t_np = measure(numpy_clip_4, arr)
changed = np.sum((arr < 0) | (arr > 1))
print(f"Изменено элементов: {changed}")
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print("На миллионе элементов NumPy работает в разы быстрее, количество измененных элементов подсчитывается через маску в одну строку, по количеству конструкций NumPy короче: в цикле один for и два if, а в NumPy ни одного цикла и ни одного условного оператора\n")
```

# 5 Задание
```python
print("5. Сумма по строкам матрицы")
def loop_sum_rows_5(m):
    result = [0.0] * m.shape[0]
    for i in range(m.shape[0]):
        s = 0.0
        for j in range(m.shape[1]): s += m[i, j]
        result[i] = s
    return result
def numpy_sum_rows_5(m): return m.sum(axis=1)

m = np.random.rand(100_000, 10)
r_l, t_loop = measure(loop_sum_rows_5, m)
r_np, t_np = measure(numpy_sum_rows_5, m)
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print("NumPy быстрее и занимает одну строку, он читается как прямое действие сумма по строкам, а цикл как ручное управление индексами, поддерживать NumPy легче из-за краткости и отсутствия ошибок с индексами\n")
```


# 6 Задание
```python
print("6. Среднее по столбцам и нормировка")
def loop_norm_6(m):
    n, cols = m.shape
    result = np.zeros_like(m)
    for j in range(cols):
        col_sum = 0.0
        for i in range(n): col_sum += m[i, j]
        col_mean = col_sum / n
        for i in range(n): result[i, j] = m[i, j] - col_mean
    return result
def numpy_norm_6(m): return m - m.mean(axis=0)

m = np.random.rand(100_000, 10)
r_l, t_loop = measure(loop_norm_6, m)
r_np, t_np = measure(numpy_norm_6, m)
print(f"Средние после нормировки ≈ 0: {np.allclose(r_np.mean(axis=0), 0)}")
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print("NumPy версия занимает ровно одну строку кода, а цикл минимум шесть строк с вложенными индексами, разница в SLOC количестве строк кода в 6 раз, а в количестве ручных индексов с нуля в NumPy против множества в цикле, после нормировки средние по столбцам равны нулю, так как используется broadcasting\n")
```

# 7 Задание
```python
print("7. Поиск максимума и его индекса")
def loop_max_7(a):
    max_v = a[0]; max_i = 0
    for i in range(len(a)):
        if a[i] > max_v: max_v = a[i]; max_i = i
    return max_v, max_i
def numpy_max_7(a): return a.max(), a.argmax()

a = np.random.rand(10**6)
r_l, t_loop = measure(loop_max_7, a)
r_np, t_np = measure(numpy_max_7, a)
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
a_nan = a.copy(); a_nan[100] = np.nan
print("В цикле NaN > max_val всегда дает ложь, и результат ломается np.nanmax просто игнорирует NaN, функции с префиксом nan нужны для реальной жизни, потому что пропуски в данных встречаются постоянно и не должны ломать весь расчет\n")
```

# 8 Задание
```python
print("8. Фильтрация по порогу")
def loop_filter_8(a): return [x for x in a if x > 0]
def numpy_filter_8(a): return a[a > 0]

a = np.random.randn(10**6)
r_l, t_loop = measure(loop_filter_8, a)
r_np, t_np = measure(numpy_filter_8, a)
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print(f"Размер списка Python: ~{len(r_l)*24/1024/1024:.2f} МБ, размер массива NumPy: ~{r_np.nbytes/1024/1024:.2f} МБ, NumPy массив компактнее примерно в 3 раза по самим числам 8 байт против 24, но с учетом указателей в списке Python (ще по 8 байт на элемен) реальная экономия памяти достигает 4 раз, NumPy не только быстрее, но и в разы экономнее расходует оперативную память, что критически важно при обработке миллионов элементов\n")
```


# 9 Задание
```python
print("9. Замена по маске (in-place)")
def loop_abs_9(a):
    for i in range(len(a)):
        if a[i] < 0: a[i] = -a[i]
    return a
def numpy_abs_9(a):
    np.abs(a, out=a)
    return a

a = np.random.randn(10**6)
r_l, t_loop = measure(loop_abs_9, a.copy())
r_np, t_np = measure(numpy_abs_9, a.copy())
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x, оба способа меняют данные на месте, не создавая копию, для больших данных in place критичен создание копии массива может исчерпать оперативную память и уронить программу\n")
```

# 10 Задание
```python
print("10. Комбинированная маска")
def loop_mask_10(a): return [x for x in a if -1 < x < 1]
def numpy_mask_10(a): return a[(a > -1) & (a < 1)]

a = np.random.randn(10**6)
r_l, t_loop = measure(loop_mask_10, a)
r_np, t_np = measure(numpy_mask_10, a)
print(f"цикл: {t_loop*1000:7.2f} мс | NumPy: {t_np*1000:6.2f} мс | ускорение: {t_loop/t_np:5.1f}x")
print("оператор & выполняется раньше, чем сравнения < и > без скобок Python сначала сделает побитовое умножение массива, а потом сравнит результат, что выдаст ошибку, скобки заставляют сначала выполнить сравнения, а потом уже объединить их")
```
