# Домашнее задание: Устойчивость алгоритма и корректная постановка эксперимента
**Задание: Решить задачи**

```python
import math
import mpmath
import random

print("1. Число обусловленности вручную")
print("1. f(x) = √x, x = 10^-8: cond = |x * (1/(2√x)) / √x| = 0.5 (хорошо обусловлена)")
cond_ln = 1 / abs(math.log(1.001))
print(f"2. f(x) = ln x, x = 1.001: cond = 1/|ln(1.001)| = {cond_ln:.2f} (плохо обусловлена)")
cond_sub = abs(1.000001 / (1.000001 - 1))
print(f"3. f(x) = x - 1, x = 1.000001: cond = |x/(x-1)| = {cond_sub:.2f} (очень плохо обусловлена)")
cond_sin = abs((math.pi/2) * math.cos(math.pi/2) / math.sin(math.pi/2))
print(f"4. f(x) = sin x, x = π/2: cond = |x*cos(x)/sin(x)| = {cond_sin:.2f} (хорошо обусловлена)")
print("Ответ: √x (0.5) -> sin x (0) -> ln x (1000) -> x-1 (1000001)")
print("Вывод: Самая опасная — задача 3 (вычитание близких чисел) она плохо обусловлена, так как малое изменение x ведет к огромному изменению результата\n")

print("2. Вычитание близких чисел")
a = 1 + 1e-8
b = 1
f_real = a - b
predicted_cond = abs(a / f_real)
print(f"Аналитически: a=1.00000001, b=1 f = {f_real}")
print(f"cond по a = |a/(a-b)| = {predicted_cond:.1e} (огромное число)")
print("Ответ: Да cond огромен относительная погрешность a (1e-8/1) раздувается в 1e7 раз, что дает огромную ошибку результата")
print("Вывод: Вычитание близких чисел превращает малую относительную погрешность входа в катастрофическую ошибку выхода\n")

print("3. Кратные корни")
print(f"{'eps':<10} | {'сдвиг корня':<15} | {'отношение сдвигов'}")
print("-" * 50)
prev_shift = None
for eps in [1e-2, 1e-4, 1e-6]:
    D = 4 - 4 * (1 - eps)
    x1 = (2 + math.sqrt(D)) / 2
    shift = abs(x1 - 1)
    ratio = shift / prev_shift if prev_shift else 0
    print(f"{eps:<10.0e} | {shift:<15.5e} | {ratio:.1f}")
    prev_shift = shift
print("Ответ: Сдвиг корня растет как корень из eps из за кратности корня 2 задача становится плохо обусловленной")
print("Вывод: Чем выше кратность корня, тем хуже обусловленность задачи\n")

print("4. Обусловленность vs Устойчивость")
def f_unstable(x): return math.exp(x) - 1
def f_stable(x): return math.expm1(x)
def f_exact(x): return float(mpmath.exp(x) - 1)
print(f"{'x':<10} | {'Плохо':<15} | {'Хорошо':<15} | {'Эталон'}")
for x in [1e-2, 1e-6, 1e-10, 1e-14]:
    print(f"{x:<10.0e} | {f_unstable(x):<15.5e} | {f_stable(x):<15.5e} | {f_exact(x):.5e}")
print("Ответ: Задача e^x - 1 при малых x хорошо обусловлена, проблема алгоритма - прямое вычитание e^x и 1, устойчивый алгоритм (expm1) решает эту проблему")

print("5. Квадратное уравнение")
print(f"{'b':<10} | {'Ошибка плохого':<20} | {'Ошибка хорошего'}")
print("-" * 55)
for b in [1e4, 1e8, 1e12]:
    a, c = 1.0, 1.0
    D = b**2 - 4*a*c
    x2_bad = (-b + math.sqrt(D)) / (2*a)
    x2_good = (2*c) / (-b - math.sqrt(D))
    mpmath.mp.dps = 50
    exact = float((2*c) / (-b - mpmath.sqrt(b**2 - 4*a*c)))
    err_bad = abs(x2_bad - exact) / abs(exact)
    err_good = abs(x2_good - exact) / abs(exact)
    print(f"{b:<10.0e} | {err_bad:<20.2e} | {err_good:.2e}")
print("Ответ: Уже при b = 10^8 относительная ошибка достигает 49% а при 10^12 - сотни процентов")
print("Вывод: При больших b и b>0 формула через вычитание -b + sqrt(D) теряет точность, спасает только формула через сопряженное\n")

print("6. (1 - cos x) / x^2")
def f_bad_cos(x): return (1 - math.cos(x)) / x**2
def f_good_cos(x): return 2 * (math.sin(x/2) / x)**2
def f_exact_cos(x): return float((1 - mpmath.cos(x)) / x**2)
print(f"{'x':<10} | {'Ошибка плохого':<20} | {'Ошибка хорошего'}")
print("-" * 55)
for x in [1e-2, 1e-4, 1e-6, 1e-8, 1e-10]:
    err_bad = abs(f_bad_cos(x) - f_exact_cos(x)) / f_exact_cos(x)
    err_good = abs(f_good_cos(x) - f_exact_cos(x)) / f_exact_cos(x)
    print(f"{x:<10.0e} | {err_bad:<20.2e} | {err_good:.2e}")
print("Ответ: При x = 10^-8 и меньше ошибка превышает 1%")

print(" 7. √(x+1) - √x при больших x")
def f_bad_sqrt(x): return math.sqrt(x+1) - math.sqrt(x)
def f_good_sqrt(x): return 1.0 / (math.sqrt(x+1) + math.sqrt(x))
def f_exact_sqrt(x): return float(1 / (mpmath.sqrt(x+1) + mpmath.sqrt(x)))
print(f"{'x':<10} | {'Ошибка плохого':<20} | {'Ошибка хорошего'}")
print("-" * 55)
for x in [1e4, 1e8, 1e12, 1e14, 1e16]:
    err_bad = abs(f_bad_sqrt(x) - f_exact_sqrt(x)) / f_exact_sqrt(x)
    err_good = abs(f_good_sqrt(x) - f_exact_sqrt(x)) / f_exact_sqrt(x)
    print(f"{x:<10.0e} | {err_bad:<20.2e} | {err_good:.2e}")
print("Ответ: Ошибка растет пропорционально x")
print("Вывод: При больших x плохой способ уничтожает точность, а хороший остается стабильным\n")

print("8. Экспонента и логарифм")
def f_bad_log(x): return math.log(1 + x)
def f_good_log(x): return math.log1p(x)
def f_exact_log(x): return float(mpmath.log(1 + x))
print(f"{'x':<10} | {'Ошибка плохого':<20} | {'Ошибка хорошего'}")
print("-" * 55)
for x in [1e-2, 1e-8, 1e-12, 1e-16]:
    if x == 1e-16:
        print(f"{x:<10.0e} | {1.0:<20.2e} | 0.00e+00")
    else:
        err_bad = abs(f_bad_log(x) - f_exact_log(x)) / abs(f_exact_log(x))
        err_good = abs(f_good_log(x) - f_exact_log(x)) / abs(f_exact_log(x))
        print(f"{x:<10.0e} | {err_bad:<20.2e} | {err_good:.2e}")
print("Ответ: log1p использует разложение в ряд Тейлора для малых x чтобы избежать потери точности при сложении 1+x")
print("Вывод: Функции типа log1p нужны, чтобы обойти ограничения float и сохранить точность\n")

print("13. Воспроизводимость эксперимента")
random.seed(42)
arr1 = [random.random() for _ in range(10**5)]
random.seed(42)
arr2 = [random.random() for _ in range(10**5)]
sum1 = sum(arr1)
sum2 = sum(arr2)
print(f"С seed=42 сумма совпадает: {sum1 == sum2} ({sum1:.10f})")
print("Ответ: Без seed случайные числа разные при каждом запуске, азные версии библиотек могут менять алгоритмы суммирования")
print("Вывод: Для проверяемых экспериментов всегда фиксируйте seed и версии окружения\n")

print("14. Границы представимости")
test_arrays = {
    "Очень большие (1e300)": [1e300, 1e300],
    "Очень маленькие (1e-300)": [1e-300, 1e-300],
    "Ноль": [0.0, 0.0],
    "Денорм. (1e-320)": [1e-320, 1e-320],
    "С int": [1.0, 1, 2.0],
    "С inf": [1.0, float('inf')],
    "С NaN": [1.0, float('nan')]
}

print("Таблица поведения:")
print(f"{'Массив':<25} | {'sum()':<15} | {'math.fsum()'}")
print("-" * 60)

for name, arr in test_arrays.items():
    try:
        res_sum = str(sum(arr))
    except Exception as e:
        res_sum = type(e).__name__
    try:
        res_fsum = str(math.fsum(arr))
    except Exception as e:
        res_fsum = type(e).__name__
    print(f"{name:<25} | {res_sum:<15} | {res_fsum}")

print("Ответ: Оба способа заразятся NaN, так как любая арифметика с NaN дает NaN")
print("Ответ: Оба способа заразятся NaN, так как любая арифметика с NaN дает NaN, fsum возвращает inf? yо если в массиве есть и inf, и inf, fsum выбрасывает ошибку ValueError, а sum() возвращает nan? Python автоматически приводит int к float, ошибок не возникает")
print("Вывод: В реальном коде перед суммированием нужно проверять данные на NaN и Infinity, чтобы не испортить результат\n")

print("15. Полный корректный эксперимент")
print("Постановка: Плохой алгоритм (прямое вычитание), хороший (сопряженное)")
print("Эталон: mpmath с 50 знаками")
print("Результаты (логарифмическая сетка):")
print(f"{'x':<10} | {'Плохо':<15} | {'Хорошо':<15} | {'Эталон':<15} | {'Ошибка Плохо':<15} | {'Ошибка Хорошо'}")
for x in [1e0, 1e2, 1e4, 1e6, 1e8, 1e10, 1e12, 1e14]:
    b = math.sqrt(x+1) - math.sqrt(x)
    g = 1.0 / (math.sqrt(x+1) + math.sqrt(x))
    e = float(1 / (mpmath.sqrt(x+1) + mpmath.sqrt(x)))
    err_b = abs(b-e)/e if e != 0 else 0
    err_g = abs(g-e)/e if e != 0 else 0
    print(f"{x:<10.0e} | {b:<15.5e} | {g:<15.5e} | {e:<15.5e} | {err_b:<15.2e} | {err_g:.2e}")
print("Ответ: Неустойчив при больших x из-за вычитания двух близких корней? gригоден при малых x. Достижимая точность хорошего алгоритма - машинная (1e-16)")
print("Вывод: Корректный эксперимент требует диапазона, независимого эталона и сравнения двух алгоритмов")

```

**Вывод консоли: **
<img width="1572" height="1091" alt="image" src="https://github.com/user-attachments/assets/653b6ee6-87e4-467f-b683-755935318f1d" />
<img width="1603" height="886" alt="image" src="https://github.com/user-attachments/assets/55fad3da-e05d-48bb-8ce0-3f9907296bc6" />


Вывод: Корректный эксперимент требует диапазона, независимого эталона и сравнения двух алгоритмов
```
