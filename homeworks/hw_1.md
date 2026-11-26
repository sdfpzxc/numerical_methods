# Домашнее задание: Абсолютная и относительная погрешность.
## Округление и накопление ошибки
**Решить задачи:**
```python
import math
import mpmath
import random
from decimal import Decimal, getcontext

print("1. Ручной счёт")
pairs = [(3.14, math.pi), (2.718, math.e), (9.8, 9.81), (0.333, 1/3)]
for x, x_exact in pairs:
    dx = abs(x - x_exact)
    dx_rel = (dx / abs(x_exact)) * 100
    print(f"x={x:<5}, dx={dx:.5e}, dx%={dx_rel:.5f}%")
print("Ответ: Для чисел порядка 1 — да. Для 9.8 и 0.333 — нет, порядки различаются, порядок относительной погрешности зависит от масштаба самого числа dx эьл абсолютная величина, dx% это безразмерная\n")

print("2. Сравнение несовместимых величин")
dx_bridge = abs(1247 - 1250)
rel_bridge = (dx_bridge / 1250) * 100
dx_bolt = abs(9.7 - 10.0)
rel_bolt = (dx_bolt / 10.0) * 100
print(f"Мост: dx={dx_bridge} м, dx%={rel_bridge:.4f}%")
print(f"Болт: dx={dx_bolt} мм, dx%={rel_bolt:.4f}%")
print("Ответ: Инженер, измерявший болт 3% против 0.24% у моста, абсолютная у моста больше, но мост огромный, для болта 0.3 мм - это 3% относ погрешность уравнивает масштабы\n")

print("3. Функция relative_error")
def relative_error(approx, exact):
    if exact == 0:
        return float('inf')
    return abs(approx - exact) / abs(exact)

for a, e in [(3.14, math.pi), (2.718, math.e), (9.8, 9.81), (0.333, 1/3)]:
    print(f"rel_err({a}, {e:.4f}) = {relative_error(a, e):.5e}")
print(f"Ответ: Нет, защита возвращает {relative_error(5, 0)} при делении на ноль\n")

print("4. Обратная задача")
x_approx = 47.3
delta_x = x_approx * 0.001
print(f"Диапазон: [{x_approx - delta_x:.4f}, {x_approx + delta_x:.4f}]")
print("Ответ: [47.25, 47.35] Разумное число значащих цифр - 4\n")

print("5. Порог по точности")
T_max = 0.3 / 0.005
print(f"Порог T = {T_max}")
print("Ответ: При T <= 60 градусовб, ошибка А постоянна 0.3, ошибка Б растет с T 0.5%, малые T - выгоднее А, большие - Б\n")

print("6. Подсчёт значащих цифр")
nums_sci = [0.004520, 1200, 1.200, 10.03, 0.03050, 250000]
for n in nums_sci:
    print(f"{n:>8} -> {n:.3e}")
print("Ответ: Для 1200 и 250000, пепонятно, являются ли нули на конце значащими цифрами или просто заполнителями разрядов\n")

print("7. Честная запись результата")
print("Ответ: Правильная запись: 4.74 +- 0.02 Осталось 3 значащие цифры 4, 7, 4, Нельзя давать точность результата выше точности погрешности, последняя цифра результата и погрешности должны быть в одном разряде сотые\n")

print("8. Округление и banker's rounding")
print(f"round(0.5)={round(0.5)}, round(1.5)={round(1.5)}, round(2.5)={round(2.5)}")
print("Ответ: Python использует банковское округление к ближайшему четному, а не всегда вверх")
print("Вывод: При миллионах операций школьное округление вверх даст систематический перекос суммы, а банковское нет\n")

print("9. Промежуточное округление")
sum_a = sum(1.0 / k for k in range(1, 1001))
sum_b = sum(round(1.0 / k, 3) for k in range(1, 1001))
exact_sum = math.fsum(1.0 / k for k in range(1, 1001))
ds_a = abs(sum_a - exact_sum)
ds_b = abs(sum_b - exact_sum)
print(f"Delta S (без окр) = {ds_a:.5e}, Delta S (с окр) = {ds_b:.5e}")

if ds_a == 0:
    print("Ответ: Ошибка вар а равна 0, тк sum() случайно совпал с точным fsum вар б содержит ошибку")
else:
    print(f"Ответ: В {ds_b / ds_a:.1f} раз")
print("Вывод: Промежуточные результаты округлять нельзя, это копит ошибку\n")

print("10. Классика 0.1 + 0.2")
print(f"0.1 + 0.2 = {0.1 + 0.2:.20f}")
print("Ответ: Компьютер хранит числа в двоичной системе, дробь 1/10 превращается в бесконечную периодическую дробь, память ограничена, поэтому компьютер обрезает эту бесконечность, и получается крошечная ошибка\n")

print("11. Катастрофическая потеря точности")
print(f"a - b = {(1 + 1e-15) - 1} Относ погрешность = 1.0")
D = 1e9**2 - 4*1*1
x1_bad = (-1e9 + math.sqrt(D)) / 2
x1_good = (2*1) / (-1e9 - math.sqrt(D))
print(f"x1 (плохо) = {x1_bad}, x1 (хорошо) = {x1_good}")
print("Ответ: Через прямое вычитание - 0.0 (неверно) через сопряженное - точно (1e-9)\n")

print("12. Накопление в цикле")
arr = [0.1] * 1_000_000
sum_loop = 0.0
for x in arr: sum_loop += x
print(f"sum(): ошибка {abs(sum(arr) - 100000.0):.2e}, fsum(): ошибка {abs(math.fsum(arr) - 100000.0):.2e}")
print("Ответ: fsum точнее , тк компенсирует потерю битов\n")

print("13. Суммирование в правильном порядке")
small = [1e-10] * 500_000
large = [1e6] * 500_000
sum_sf = sum(small + large)
sum_lf = sum(large + small)
print(f"(a) малые сначала: {sum_sf:.5e}, (b) большие сначала: {sum_lf:.5e}")
print("Ответ: При прибавлении маленького числа к большому, маленькое теряется, лучше складывать от меньшего к большему\n")

print("14. Детектор накопления погрешности")
x = 1.0
th = {1e3: None, 1e6: None, 1e9: None}
for i in range(1, 10001):
    x *= 1.0001
    err = abs(x - math.exp(i * math.log(1.0001))) / math.exp(i * math.log(1.0001))
    for t in th:
        if err > t and th[t] is None:
            th[t] = i
print(f"Ответ: 10^3 на шаге {th[1e3]}, 10^6 на шаге {th[1e6]}, 10^9 на шаге {th[1e9]}")
print("Вывод: В мультипликативных процессах ошибка раздувается экспоненциально\n")

print("15. Финансы: float vs Decimal")
getcontext().prec = 28
acc_f = 0.0; acc_d = Decimal('0.0'); acc_i = 0
for _ in range(1_000_000):
    acc_f += 0.01; acc_d += Decimal('0.01'); acc_i += 1
print(f"Float: {acc_f:.10f}, Decimal: {acc_d}, Int: {acc_i / 100:.2f}")

print("Ответ: decimal и int, float накапливает ошибку, бухгалтерии копейки должны сходиться идеально, иначе недостачи/излишки\n")

print("16. Мини-исследование")
arr = [random.random() for _ in range(10**6)]
f_m = 2.2e-16
m_fsum = math.fsum(arr) / 10**6
m_sum = sum(arr) / 10**6
print(f"Отклонение sum(): {abs(m_sum - m_fsum)/f_m:.1f} * e_machine, fsum(): 0.0 (эталон)")
print("Ответ: Точнее всех math.fsum, numpy хорош парным суммированием, но математически не всегда гарантирует идеальный результат")

import time

# "It's Raining Tacos Again" by Parry Gripp
# Lyrics provided by user, typed letter-by-letter.

song = [
    "It's raining tacos again",
    "They're coming down and everyone wins",
    "Oh, people are a-crunching and a-munching all across the land",
    "Shell, meat and lettuce and cheese",
    "You can eat as many as you please (Yes please)",
    "Oh, everybody's mowing, the salsa will be flowing",
    "'Cause it's raining tacos again",
    "Every day is taco day",
    "Forever and ever and ever and ever",
    "You wonder when you'll be hungry again",
    "The answer is never, never, never again",
    "",
    "Everyone gets tacos",
    "Everyone gets tacos",
    "Tacos, tacos, everyone gets tacos",
    "(Shell) Shell (Meat) Meat (Lettuce) Lettuce (Cheese) Cheese",
    "Everyone gets tacos",
    "Everyone gets tacos",
    "",
    "It's raining tacos again (Ooh)",
    "It's raining tacos again (Ooh)",
    "Oh, tacos, tacos",
    "Shell, meat and lettuce and cheese",
    "(Cheese, cheese, cheese, cheese)",
    "It's raining tacos again (Ooh)",
    "It's raining tacos again (Everyone gets tacos)",
    "Oh, tacos, tacos, tacos, tacos",
    "Tacos, tacos, tacos",
    "It's raining tacos again (Cheese, cheese, cheese, cheese)",
]

# Typing speed: seconds per letter
LETTER_DELAY = 0.05
# Pause after finishing a full line
LINE_PAUSE = 0.3


def type_line(text, delay=LETTER_DELAY):
    """Print a line letter by letter."""
    if text == "":
        print()
        return

    for char in text:
        print(char, end="", flush=True)
        time.sleep(delay)
    print()


def play_song():
    print("=" * 60)
    print("🌮 IT'S RAINING TACOS AGAIN - Parry Gripp 🌮".center(60))
    print("=" * 60)
    print()

    for line in song:
        type_line(line)
        time.sleep(LINE_PAUSE)


if __name__ == "__main__":
    try:
        play_song()
    except KeyboardInterrupt:
        print("\n\n⏹  Stopped by user.")
```
**Вывод консоли:**
<img width="1569" height="1112" alt="image" src="https://github.com/user-attachments/assets/5ff7a52e-6606-4cc8-b569-c5c405437e86" />
<img width="1604" height="1141" alt="image" src="https://github.com/user-attachments/assets/4f810e3d-fe11-46a8-9b19-c4d86af58bb6" />

