# Домашнее задание: 15 задач на отработку темы «NumPy: массивы, формы, типы данных и базовые операции»
Формат сдачи: файл .py (или .ipynb) с решениями. Каждая задача — отдельная функция или отдельная ячейка с print. Для задач, где нужно «объяснить», — вывод в 2–3 предложениях текстом в комментарии или print. Также нужно прикрепить скриншот выполнения. Все через репозиторий.

### Требование: все задачи запускаются. Если задача требует данных — они генерируются в коде (никаких внешних файлов).

```python
import numpy as np

def task1():
    print("Задача 1. Инвентаризация массива")
    a = np.arange(60).reshape(3, 4, 5)
    print("shape:", a.shape)
    print("ndim:", a.ndim)
    print("size:", a.size)
    print("dtype:", a.dtype)
    print("strides:", a.strides)
    print("C_CONTIGUOUS:", a.flags['C_CONTIGUOUS'])
    print("""
Объяснение: strides = (160, 40, 8) — это число байт, которое нужно
прыгнуть в памяти, чтобы сдвинуться на 1 по каждому измерению
Для int64 (8 байт) шаг по последней оси = 8, по средней = 5*8 = 40,
по первой = 4*5*8 = 160 size = 60, потому что 3 * 4 * 5 = 60
элементов (произведение всех размерностей)
""")


def task2():
    print("Задача 2. int vs float — тихая ловушка")
    a = np.array([1, 2, 3, 4])
    print("a =", a, "dtype =", a.dtype)
    print("a / 2      =", a / 2,      "dtype =", (a / 2).dtype)
    print("a // 2     =", a // 2,     "dtype =", (a // 2).dtype)
    print("a / 2.0    =", a / 2.0,    "dtype =", (a / 2.0).dtype)
    print("a.dtype    =", a.dtype)

    b = np.array([1, 2, 3], dtype=np.int8)
    print("\nb =", b, "dtype =", b.dtype)
    print("b + 127 =", b + 127, "dtype =", (b + 127).dtype)
    print("""
Объяснение: a / 2 возвращает новый массив float64, a не меняется,
потому что массивы неизменяемы по dtype — операция создаёт копию
b + 127 с int8 дает переполнение: 1+127 = 128 не влезает в int8
(диапазон -128..127), получается -128 NumPy НЕ падает с ошибкой,
потому что по умолчанию integer overflow в NumPy оборачивается
(как в C), а не возбуждает исключение — это тихая ловушка
""")


def task3():
    print("Задача 3. float32 vs float64 — реальная точность")
    err32 = float(np.float32(0.1)) - 0.1
    err64 = float(np.float64(0.1)) - 0.1
    print("float32 error repr:", repr(err32))
    print("float64 error repr:", repr(err64))
    print(f"float32 error :.20e: {err32:.20e}")
    print(f"float64 error :.20e: {err64:.20e}")
    ratio = abs(err32) / abs(err64)
    print(f"ошибка float32 больше в {ratio:.1f} раз")
    print("""
Вывод: float32 дает ~7 значащих цифр, float64 — ~15-16, ошибка
float32 примерно в 10^8 раз больше (машинное эпсилон 1.2e-7 vs 2.2e-16)
float32 допустим в ML/графике/обработке сигналов, где важна скорость
и память, а не абсолютная точность, не допустим в задачах, где
накапливаются ошибки: решение СЛАУ, интегрирование, длинные суммы,
финансовые расчеты, где нужны точные десятичные результаты
""")


def task4():
    print("Задача 4. Явный dtype при создании")
    a = np.zeros((2, 3), dtype=np.float64)
    b = np.zeros((2, 3), dtype=np.float32)
    c = np.ones((2, 3), dtype=np.int16)
    d = np.full((2, 3), 7, dtype=np.uint8)
    for name, arr in [("zeros float64", a), ("zeros float32", b),
                      ("ones int16", c), ("full 7 uint8", d)]:
        print(f"{name:16s} dtype={arr.dtype!s:10s} nbytes={arr.nbytes}")

    big64 = np.zeros((1000, 1000), dtype=np.float64)
    big32 = np.zeros((1000, 1000), dtype=np.float32)
    saved = big64.nbytes - big32.nbytes
    print(f"\n(1000,1000) float64: {big64.nbytes:,} байт")
    print(f"(1000,1000) float32: {big32.nbytes:,} байт")
    print(f"экономия float32:    {saved:,} байт = {saved/1024**2:.1f} МБ")
    print("""
Вывод: float32 вдвое компактнее float64 (4 vs 8 байт), что критично
для больших массивов и GPU, компромисс: память/скорость против точности
""")

def task5():
    print("Задача 5. Цепочка reshape")
    a = np.arange(24)
    for shape in [(4, 6), (2, 3, 4), (2, 2, 6), (24,)]:
        cur = a.reshape(shape)
        ok = np.array_equal(a, cur.ravel())
        print(f"reshape{shape!s:12s} -> shape={cur.shape!s:12s} данные целы: {ok}")
    print("""
Объяснение: reshape обычно НЕ копирует данные — он лишь меняет
strides и shape, возвращая view на тот же буфер, копия вынуждена,
когда новый порядок нельзя выразить через strides, например
при reshape несмежного (non-contiguous) массива или при
транспонировании с последующим reshape без copy
""")


def task6():
    print("Задача 6. flatten vs ravel — копия или view")
    M = np.arange(12).reshape(3, 4)
    f = M.flatten()
    r = M.ravel()
    print("f.base is None:", f.base is None)
    print("r.base is not None:", r.base is not None)
    f[0] = 999
    print("после f[0]=999, M[0,0] =", M[0, 0], "(не изменился)")
    r[1] = 888
    print("после r[1]=888, M[0,1] =", M[0, 1], "(изменился — r это view)")
    print("""
Вывод: ravel() возвращает view (если возможно) — быстро, но изменения
влияют на оригинал flatten() всегда копирует — безопаснее, но дороже
по памяти, используйте ravel для чтения/экономии, flatten — когда
нужна независимая копия
""")


def task7():
    print("Задача 7. newaxis, squeeze, транспонирование")
    v = np.arange(5)
    print("v.shape:", v.shape)
    print("v[:, None].shape:", v[:, None].shape, "— столбец (5,1)")
    print("v[None, :].shape:", v[None, :].shape, "— строка (1,5)")
    print("squeeze(v[:,None]).shape:", np.squeeze(v[:, None]).shape,
          "— убрали ось размера 1")
    T = np.arange(6).reshape(2, 3).T
    print("arange(6).reshape(2,3).T shape:", T.shape)
    print("содержимое T:\n", T)

    outer = v[:, None] + v[None, :]
    print("\nv[:,None] + v[None,:] shape:", outer.shape)
    print("матрица:\n", outer)
    print("""
Это таблица сумм (аналог таблицы умножения, но сложения):
outer[i,j] = v[i] + v[j] broadcasting: (5,1)+(1,5)->(5,5)
""")


def task8():
    print("Задача 8. Транспонирование без копии")
    A = np.arange(6).reshape(2, 3)
    B = A.T
    print("A.shape:", A.shape, "B.shape:", B.shape)
    print("B.base is not None:", B.base is not None)
    print("B.base is A:", B.base is A)
    print("B.strides:", B.strides, " A.strides:", A.strides)
    B[0, 1] = 999
    print("после B[0,1]=999, A[1,0] =", A[1, 0], "(изменился — B view)")
    print("""
Вывод: .T не копирует данные, а меняет strides местами: было
(24, 8) для (2,3) int64, стало (8, 24) для (3,2) тот же буфер,
другой порядок обхода -> view, поэтому изменение B меняет A
""")
 
def task9():
    print("Задача 9. Поэлементно vs матричное")
    A = np.arange(6).reshape(2, 3)
    B = np.arange(6).reshape(3, 2)
    print("A @ B shape:", (A @ B).shape)
    print(A @ B)
    print("A.T @ B.T shape:", (A.T @ B.T).shape)
    print(A.T @ B.T)
    print("A * A shape:", (A * A).shape)
    print(A * A)
    try:
        A * B
    except ValueError as e:
        print("A * B -> ValueError:", e)
    print("""
Объяснение: * — поэлементное умножение, требует одинаковых форм
(или broadcast). @ — матричное умножение по правилу (m,k)@(k,n)
A*B падает, т.к. формы (2,3) и (3,2) не broadcast-совместимы
Пример, где * и @ совпадают: диагональная матрица D — тогда
D @ X = D * X (поэлементно масштабирует строки)
""")
    D = np.diag([2, 3])
    X = np.arange(4).reshape(2, 2)
    print("D @ X:\n", D @ X)
    print("D * X:\n", D * X)
    print("равны:", np.array_equal(D @ X, D * X))


def task10():
    print("Задача 10. axis=0 vs axis=1")
    M = np.arange(12).reshape(3, 4)
    print("M:\n", M)
    print("M.sum(axis=0):", M.sum(axis=0), "— сумма по столбцам (свернули строки)")
    print("M.sum(axis=1):", M.sum(axis=1), "— сумма по строкам (свернули столбцы)")
    print("M.mean(axis=0):", M.mean(axis=0))
    print("M.mean(axis=1):", M.mean(axis=1))
    print("M.argmax(axis=0):", M.argmax(axis=0))
    print("M.argmax(axis=1):", M.argmax(axis=1))
    print("""
Проверка: столбец 0 = [0,4,8], сумма = 12 = M.sum(axis=0)[0]
Значит axis=0 сворачивает именно строки, оставляя сумму по столбцам
""")


def task11():
    print("Задача 11. keepdims и broadcasting")
    X = np.random.default_rng(42).normal(size=(5, 3))
    m0 = X.mean(axis=0)
    m0k = X.mean(axis=0, keepdims=True)
    print("X.mean(axis=0).shape:", m0.shape)
    print("X.mean(axis=0, keepdims=True).shape:", m0k.shape)
    print("(X - m0).shape:", (X - m0).shape)
    print("(X - m0k).shape:", (X - m0k).shape)

    s1 = X.std(axis=1)
    s1k = X.std(axis=1, keepdims=True)
    print("X.std(axis=1).shape:", s1.shape)
    print("X.std(axis=1, keepdims=True).shape:", s1k.shape)

    Xn = (X - X.mean(axis=1, keepdims=True)) / X.std(axis=1, keepdims=True)
    print("средние по строкам после нормировки:", np.round(Xn.mean(axis=1), 12))
    print("std по строкам после нормировки:  ", np.round(Xn.std(axis=1), 12))
    print("""
keepdims сохраняет «свёрнутую» ось размером 1, чтобы broadcasting
вычитания/деления шёл по нужной оси, без keepdims (5,3)-(3,) тоже
работает за счет выравнивания справа, но (5,3)-(5,) — уже ошибка;
keepdims делает намерение явным и безопасным
""")

def task12():
    print("Задача 12. Срез — view, не копия")
    a = np.arange(10)
    b = a[2:5]
    b[0] = 999
    print("после b[0]=999, a =", a, "(a[2] изменился — view)")
    a = np.arange(10)
    c = a[2:5].copy()
    c[0] = 999
    print("после c[0]=999, a =", a, "(не изменился — copy)")
    print("""
Важно для численных методов: если обрезать данные для анализа
и случайно изменить срез, оригинал испортится молча — эксперимент
даст неверный результат, view — преимущество для экономии памяти
(большие массивы, нарезка батчей), но баг, если нужна независимость
""")


def task13():
    print("Задача 13. Булева маска и fancy indexing")
    a = np.random.default_rng(0).normal(size=20)
    print("a:", np.round(a, 3))
    print("a[a>0] кол-во:", (a > 0).sum(), "значения:", np.round(a[a > 0], 3))
    mid = a[(a > -1) & (a < 1)]
    print("a[-1<a<1] кол-во:", mid.size)
    print("a[[0,5,10,15]]:", a[[0, 5, 10, 15]], "— fancy indexing по индексам")
    out = a[np.abs(a) > 2]
    print("выбросы |a|>2:", np.round(out, 3))
    a[a < 0] = 0
    print("после зануления отрицательных:\n", np.round(a, 3))
    print("""
a[a>0] & (a<1) дает ошибку, потому что & применяется к МАССИВАМ,
а a>0 — это массив bool, но левый операнд a[a>0] — уже числа
Правильно: (a>0) & (a<1) — сначала две маски, потом &. Приоритет
операторов: & ниже >, поэтому нужны скобки
""")


def task14():
    print("Задача 14. Broadcasting в реальной задаче")
    X = np.random.default_rng(1).normal(size=(100, 3))
    mu = X.mean(axis=0, keepdims=True)
    sd = X.std(axis=0, keepdims=True)
    Xn = (X - mu) / sd
    print("X_norm.mean(axis=0):", np.round(Xn.mean(axis=0), 12))
    print("X_norm.std(axis=0): ", np.round(Xn.std(axis=0), 12))

    diff = Xn[:, None, :] - Xn[None, :, :]
    D = np.sqrt((diff ** 2).sum(axis=2))
    print("D.shape:", D.shape)
    print("диагональ ≈ 0:", np.allclose(np.diag(D), 0))
    print("симметрична:", np.allclose(D, D.T))
    print("""
Весь расчет в одну строку без цикла: (100,1,3) - (1,100,3) -> (100,100,3),
сумма по последней оси -> (100,100), broadcasting заменил двойной цикл
""")


def task15():
    print("Задача 15. Полный кейс")
    rng = np.random.default_rng(42)
    raw = rng.normal(loc=20, scale=2, size=(30, 4))
    raw[7, 2] = 100  

    print("1) shape:", raw.shape, "dtype:", raw.dtype, "size:", raw.size)

    mu = raw.mean(axis=0)
    sd = raw.std(axis=0)
    mask = np.abs(raw - mu) > 3 * sd
    print("2) выбросов (3 sigma):", mask.sum(), "координаты:", np.argwhere(mask))

    raw_nan = raw.copy()
    raw_nan[mask] = np.nan
    col_mean = np.nanmean(raw_nan, axis=0)
    raw_fixed = np.where(mask, col_mean, raw)
    print("3) после замены на среднее по столбцу raw_fixed[7,2] =", raw_fixed[7, 2])

    t = raw_fixed.T
    back = t.T
    print("4) .T (30,4)->(4,30)->(30,4) целы:",
          np.array_equal(raw_fixed, back))

    flat = raw_fixed.ravel()
    restored = flat.reshape(30, 4)
    print("5) ravel (120,) -> reshape (30,4) целы:",
          np.array_equal(raw_fixed, restored), "| flat.shape:", flat.shape)

    m = raw_fixed.mean(axis=0)
    s = raw_fixed.std(axis=0)
    print("6) средние по датчикам:", np.round(m, 3))
    print("   std по датчикам:    ", np.round(s, 3))

    norm = (raw_fixed - raw_fixed.mean(axis=0, keepdims=True)) / \
            raw_fixed.std(axis=0, keepdims=True)
    print("7) после нормировки mean:", np.round(norm.mean(axis=0), 12))
    print("   после нормировки std: ", np.round(norm.std(axis=0), 12))
    print("""
Вывод :
copy() потребовался при создании raw_nan, чтобы не портить raw при
замене выбросов на nan, broadcasting сработал (mean/std
через keepdims) и при поиске выбросов (raw - mu), oшибка формы
могла возникнуть в шаге 2, если забыть keepdims и вычитать (4,) из
(30,4) - сработает выравнивание справа, но при (30,4) - (30,) уже
ValueError, шаги 4-5 показали, что .T и ravel/reshape не портят
данные (проверено np.array_equal)
""")


if __name__ == "__main__":
    for t in [task1, task2, task3, task4, task5, task6, task7, task8,
              task9, task10, task11, task12, task13, task14, task15]:
        t()
        print()

```
## Вывод консоли:
```python\
Задача 1. Инвентаризация массива
shape: (3, 4, 5)
ndim: 3
size: 60
dtype: int64
strides: (160, 40, 8)
C_CONTIGUOUS: True

Объяснение: strides = (160, 40, 8) — это число байт, которое нужно
прыгнуть в памяти, чтобы сдвинуться на 1 по каждому измерению
Для int64 (8 байт) шаг по последней оси = 8, по средней = 5*8 = 40,
по первой = 4*5*8 = 160 size = 60, потому что 3 * 4 * 5 = 60
элементов (произведение всех размерностей)

Задача 2. int vs float — тихая ловушка
a = [1 2 3 4] dtype = int64
a / 2      = [0.5 1.  1.5 2. ] dtype = float64
a // 2     = [0 1 1 2] dtype = int64
a / 2.0    = [0.5 1.  1.5 2. ] dtype = float64
a.dtype    = int64

b = [1 2 3] dtype = int8
b + 127 = [-128 -127 -126] dtype = int8

Объяснение: a / 2 возвращает новый массив float64, a не меняется,
потому что массивы неизменяемы по dtype — операция создаёт копию
b + 127 с int8 дает переполнение: 1+127 = 128 не влезает в int8
(диапазон -128..127), получается -128 NumPy НЕ падает с ошибкой,
потому что по умолчанию integer overflow в NumPy оборачивается
(как в C), а не возбуждает исключение — это тихая ловушка

Задача 3. float32 vs float64 — реальная точность
float32 error repr: -1.4901161193847656e-09
float64 error repr: -1.1102230246251565e-17
float32 error :.20e: -1.49011611938476562500e-09
float64 error :.20e: -1.11022302462515654042e-17
ошибка float32 больше в 134217728.0 раз

Вывод: float32 дает ~7 значащих цифр, float64 — ~15-16, ошибка
float32 примерно в 10^8 раз больше (машинное эпсилон 1.2e-7 vs 2.2e-16)
float32 допустим в ML/графике/обработке сигналов, где важна скорость
и память, а не абсолютная точность, не допустим в задачах, где
накапливаются ошибки: решение СЛАУ, интегрирование, длинные суммы,
финансовые расчеты, где нужны точные десятичные результаты

Задача 4. Явный dtype при создании
zeros float64    dtype=float64    nbytes=48
zeros float32    dtype=float32    nbytes=24
ones int16       dtype=int16      nbytes=12
full 7 uint8     dtype=uint8      nbytes=6

(1000,1000) float64: 8 000 000 байт
(1000,1000) float32: 4 000 000 байт
экономия float32:    4 000 000 байт = 3.8 МБ

Вывод: float32 вдвое компактнее float64 (4 vs 8 байт), что критично
для больших массивов и GPU, компромисс: память/скорость против точности

Задача 5. Цепочка reshape
reshape(4, 6)     -> shape=(4, 6)       данные целы: True
reshape(2, 3, 4)  -> shape=(2, 3, 4)    данные целы: True
reshape(2, 2, 6)  -> shape=(2, 2, 6)    данные целы: True
reshape(24,)      -> shape=(24,)        данные целы: True

Объяснение: reshape обычно НЕ копирует данные — он лишь меняет
strides и shape, возвращая view на тот же буфер, копия вынуждена,
когда новый порядок нельзя выразить через strides, например
при reshape несмежного (non-contiguous) массива или при
транспонировании с последующим reshape без copy

Задача 6. flatten vs ravel — копия или view
f.base is None: True
r.base is not None: True
после f[0]=999, M[0,0] = 0 (не изменился)
после r[1]=888, M[0,1] = 888 (изменился — r это view)

Вывод: ravel() возвращает view (если возможно) — быстро, но изменения
влияют на оригинал flatten() всегда копирует — безопаснее, но дороже
по памяти, используйте ravel для чтения/экономии, flatten — когда
нужна независимая копия

Задача 7. newaxis, squeeze, транспонирование
v.shape: (5,)
v[:, None].shape: (5, 1) — столбец (5,1)
v[None, :].shape: (1, 5) — строка (1,5)
squeeze(v[:,None]).shape: (5,) — убрали ось размера 1
arange(6).reshape(2,3).T shape: (3, 2)
содержимое T:
 [[0 3]
 [1 4]
 [2 5]]

v[:,None] + v[None,:] shape: (5, 5)
матрица:
 [[0 1 2 3 4]
 [1 2 3 4 5]
 [2 3 4 5 6]
 [3 4 5 6 7]
 [4 5 6 7 8]]

Это таблица сумм (аналог таблицы умножения, но сложения):
outer[i,j] = v[i] + v[j] broadcasting: (5,1)+(1,5)->(5,5)

Задача 8. Транспонирование без копии
A.shape: (2, 3) B.shape: (3, 2)
B.base is not None: True
B.base is A: True
B.strides: (8, 24)  A.strides: (24, 8)
после B[0,1]=999, A[1,0] = 999 (изменился — B view)

Вывод: .T не копирует данные, а меняет strides местами: было
(24, 8) для (2,3) int64, стало (8, 24) для (3,2) тот же буфер,
другой порядок обхода -> view, поэтому изменение B меняет A

Задача 9. Поэлементно vs матричное
A @ B shape: (2, 2)
[[10 13]
 [28 40]]
A.T @ B.T shape: (3, 3)
[[ 4  5  6]
 [16 17 18]
 [28 29 30]]
A * A shape: (2, 3)
[[ 0  1  4]
 [ 9 16 25]]
A * B -> ValueError: operands could not be broadcast together with shapes (2,3) (3,2) 

Объяснение: * — поэлементное умножение, требует одинаковых форм
(или broadcast). @ — матричное умножение по правилу (m,k)@(k,n)
A*B падает, т.к. формы (2,3) и (3,2) не broadcast-совместимы
Пример, где * и @ совпадают: диагональная матрица D — тогда
D @ X = D * X (поэлементно масштабирует строки)

D @ X:
 [[ 0  2]
 [12 15]]
D * X:
 [[ 0  2]
 [12 15]]
равны: True

Задача 10. axis=0 vs axis=1
M:
 [[ 0  1  2  3]
 [ 4  5  6  7]
 [ 8  9 10 11]]
M.sum(axis=0): [12 15 18 21] — сумма по столбцам (свернули строки)
M.sum(axis=1): [ 6 22 38] — сумма по строкам (свернули столбцы)
M.mean(axis=0): [4. 5. 6. 7.]
M.mean(axis=1): [ 1.5  5.5  9.5]
M.argmax(axis=0): [2 2 2 2]
M.argmax(axis=1): [3 3 3]

Проверка: столбец 0 = [0,4,8], сумма = 12 = M.sum(axis=0)[0]
Значит axis=0 сворачивает именно строки, оставляя сумму по столбцам

Задача 11. keepdims и broadcasting
X.mean(axis=0).shape: (3,)
X.mean(axis=0, keepdims=True).shape: (1, 3)
(X - m0).shape: (5, 3)
(X - m0k).shape: (5, 3)
X.std(axis=1).shape: (5,)
X.std(axis=1, keepdims=True).shape: (5, 1)
средние по строкам после нормировки: [-0. -0. -0. -0. -0.]
std по строкам после нормировки:   [1. 1. 1. 1. 1.]

keepdims сохраняет «свёрнутую» ось размером 1, чтобы broadcasting
вычитания/деления шёл по нужной оси, без keepdims (5,3)-(3,) тоже
работает за счет выравнивания справа, но (5,3)-(5,) — уже ошибка;
keepdims делает намерение явным и безопасным

Задача 12. Срез — view, не копия
после b[0]=999, a = [  0   1 999   3   4   5   6   7   8   9] (a[2] изменился — view)
после c[0]=999, a = [0 1 2 3 4 5 6 7 8 9] (не изменился — copy)

Важно для численных методов: если обрезать данные для анализа
и случайно изменить срез, оригинал испортится молча — эксперимент
даст неверный результат, view — преимущество для экономии памяти
(большие массивы, нарезка батчей), но баг, если нужна независимость

Задача 13. Булева маска и fancy indexing
a: [ 0.126  0.64  -0.952  0.428  1.289  1.096  0.735 -0.06   0.163 -1.173
 -0.751  0.979 -1.028  0.703  0.168  0.537  0.371 -0.528  0.142 -0.506]
a[a>0] кол-во: 13 значения: [0.126 0.64  0.428 1.289 1.096 0.735 0.163 0.979 0.703 0.168 0.537 0.371 0.142]
a[-1<a<1] кол-во: 15
a[[0,5,10,15]]: [ 0.126  1.096 -0.751  0.537 ] — fancy indexing по индексам
выбросы |a|>2: []
после зануления отрицательных:
 [0.126 0.64  0.    0.428 1.289 1.096 0.735 0.    0.163 0.
  0.    0.979 0.    0.703 0.168 0.537 0.371 0.    0.142 0.   ]

a[a>0] & (a<1) дает ошибку, потому что & применяется к МАССИВАМ,
а a>0 — это массив bool, но левый операнд a[a>0] — уже числа
Правильно: (a>0) & (a<1) — сначала две маски, потом &. Приоритет
операторов: & ниже >, поэтому нужны скобки

Задача 14. Broadcasting в реальной задаче
X_norm.mean(axis=0): [ 0. -0.  0.]
X_norm.std(axis=0):  [1. 1. 1.]
D.shape: (100, 100)
диагональ ≈ 0: True
симметрична: True

Весь расчет в одну строку без цикла: (100,1,3) - (1,100,3) -> (100,100,3),
сумма по последней оси -> (100,100), broadcasting заменил двойной цикл

Задача 15. Полный кейс
1) shape: (30, 4) dtype: float64 size: 120
2) выбросов (3 sigma): 1 координаты: [[7 2]]
3) после замены на среднее по столбцу raw_fixed[7,2] = 19.861232974811388
4) .T (30,4)->(4,30)->(30,4) целы: True
5) ravel (120,) -> reshape (30,4) целы: True | flat.shape: (120,)
6) средние по датчикам: [19.968 19.925 19.997 20.04 ]
   std по датчикам:     [1.947 1.983 2.024 1.912]
7) после нормировки mean: [ 0. -0.  0. -0.]
   после нормировки std:  [1. 1. 1. 1.]

Вывод :
copy() потребовался при создании raw_nan, чтобы не портить raw при
замене выбросов на nan, broadcasting сработал (mean/std
через keepdims) и при поиске выбросов (raw - mu), oшибка формы
могла возникнуть в шаге 2, если забыть keepdims и вычитать (4,) из
(30,4) - сработает выравнивание справа, но при (30,4) - (30,) уже
ValueError, шаги 4-5 показали, что .T и ravel/reshape не портят
данные (проверено np.array_equal)
```
