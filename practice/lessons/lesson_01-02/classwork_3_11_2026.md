# 9. Мини-задание для закрепления

## Описание задания
Напиши функцию `relative_error(approx, exact)`, которая возвращает \(\delta x\), и проверь её на:
- \(\pi\) и \(3.14\);
- \((0.1 + 0.2)\) и \(0.3\) — обрати внимание, насколько результат «не 0»;
- сумме миллиона \(0.1\) против `math.fsum`.

## Решение (код на Python)

```python
import math

# Функция для вычисления относительной погрешности (δx)
def relative_error(approx, exact):
    return abs(approx - exact) / abs(exact)

print("Task 9: sad math meow")

# 1. Проверка на Пи и 3.14
dx_pi = relative_error(3.14, math.pi) 

# 2. Проверка на 0.1 + 0.2 и 0.3
dx_sum = relative_error(0.1 + 0.2, 0.3)

# 3. Проверка на сумму миллиона 0.1
list_01 = [0.1] * 1_000_000
exact_val = 100_000.0
sum_norm = sum(list_01)
sum_fsum = math.fsum(list_01)

dx_norm = relative_error(sum_norm, exact_val)
dx_fsum = relative_error(sum_fsum, exact_val)

# Вывод результатов
print("Пи (3.14):", dx_pi)
print("0.1 + 0.2:", dx_sum)
print("Миллион (sum):", dx_norm)
print("Миллион (fsum):", dx_fsum)
```
# Вывод:
- пи : Δx ~ 0.0016, δx ~ 0.05% обе погрешности маленькие
- 0.1 + 0.2: Δx ~ 5.5e-17, δx ~ 1.8e-16. обе малы но не равны 0
- sum vs math.fsum: Δx ~ 1.3e-8, δx ~ 1.3e-13 (у fsum обе равны 0) в цикле ошибка накапливается

```python
_second_part_3_11.py
Task 9: sad math meow
пи 3.14: 0.0005069573828972128
0.1 + 0.2: 1.8503717077085943e-16
миллион sum: 0.0
миллион fsum: 0.0
```
