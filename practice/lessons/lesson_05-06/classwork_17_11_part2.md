# 9. Задачи на отработку темы

## Задача 1. Поворот и отражение правильного шестиугольника
- Условие. Постройте правильный шестиугольник с центром в начале координат и радиусом 1.
- Что сдать: скрипт, график, и ответ на вопрос: почему отражённый шестиугольник выглядит «повёрнутым», хотя это отражение, а не поворот? (Подсказка: сравните матрицы поворота на 90° и отражения относительно y = x — они совпадают с точностью до знака.)

## Задача 2. Композиция преобразований и обратное преобразование
- Условие. Дан набор из 200 случайных точек, равномерно распределённых в квадрате [−2, 2] × [−2, 2] (используйте  np.random.default_rng(42) ).
- Что сдать: скрипт, график «исходные / преобразованные / восстановленные», и ответ на вопрос: почему обратные преобразования нужно применять в обратном порядке? Что произойдёт, если применить их в прямом порядке? (Подсказка: (AB)−1 = B−1 A−1

```python
import numpy as np
import matplotlib.pyplot as plt

k = np.arange(6)
hexagon = np.column_stack([np.cos(2*np.pi*k/6), np.sin(2*np.pi*k/6)])
hexagon = np.vstack([hexagon, hexagon[0]])

theta = np.deg2rad(45)
rotate45 = np.array([[np.cos(theta), -np.sin(theta)],
                     [np.sin(theta),  np.cos(theta)]])
reflect_yx = np.array([[0, 1], [1, 0]])

rotated   = hexagon @ rotate45.T
reflected = hexagon @ reflect_yx.T

fig, ax = plt.subplots(figsize=(7, 7))
ax.plot(*hexagon.T,   'o-', color='gray',      label='исходный', lw=2)
ax.plot(*rotated.T,   'o-', color='steelblue', label='поворот 45°', lw=2)
ax.plot(*reflected.T, 'o-', color='crimson',   label='отражение y=x', lw=2)
ax.axhline(0, color='black', lw=0.5); ax.axvline(0, color='black', lw=0.5)
ax.set_aspect('equal'); ax.grid(True, alpha=0.3); ax.legend()
plt.tight_layout(); plt.show()

# Ответ: R y=x = [[0,1],[1,0]], Q 90 = [[0,-1],[1,0]] — совпадают с точностью до знака
# Шестиугольник симметричен относительно y=x, поэтому отражение выглядит как поворот

rng = np.random.default_rng(42)
points = rng.uniform(-2, 2, size=(200, 2))

scale = np.array([[1.5, 0.0], [0.0, 0.5]])
theta = np.deg2rad(60)
rotate60 = np.array([[np.cos(theta), -np.sin(theta)],
                     [np.sin(theta),  np.cos(theta)]])
translate = np.array([[1, 0, 1], [0, 1, -1], [0, 0, 1]], dtype=float)

p2 = points @ (rotate60 @ scale).T
p_h = np.hstack([p2, np.ones((len(p2), 1))])
p_transformed = (translate @ p_h.T).T[:, :2]

translate_inv = np.array([[1, 0, -1], [0, 1, 1], [0, 0, 1]], dtype=float)
p_back_h = np.hstack([p_transformed, np.ones((len(p_transformed), 1))])
p_back = (translate_inv @ p_back_h.T).T[:, :2]

theta_inv = np.deg2rad(-60)
rotate_inv = np.array([[np.cos(theta_inv), -np.sin(theta_inv)],
                       [np.sin(theta_inv),  np.cos(theta_inv)]])
scale_inv = np.array([[1/1.5, 0.0], [0.0, 1/0.5]])

p_restored = p_back @ (scale_inv @ rotate_inv).T

print("Восстановление 1e-12:", np.allclose(p_restored, points, atol=1e-12))

fig, axes = plt.subplots(1, 3, figsize=(18, 6))
for ax, pts, title, c in zip(axes,
        [points, p_transformed, p_restored],
        ['Исходные', 'Преобразованные', 'Восстановленные'],
        ['gray', 'steelblue', 'crimson']):
    ax.scatter(pts[:, 0], pts[:, 1], s=14, color=c, alpha=0.7)
    ax.axhline(0, color='black', lw=0.5); ax.axvline(0, color='black', lw=0.5)
    ax.set_aspect('equal'); ax.grid(True, alpha=0.3); ax.set_title(title)
plt.tight_layout(); plt.show()

# Ответ: T = C*B*A  =>  T^{-1} = A^{-1}*B^{-1}*C^{-1}
# Обратные применяются в обратном порядке (матрицы не коммутируют)
# В прямом порядке точки не восстановятся
```

<img width="854" height="888" alt="image" src="https://github.com/user-attachments/assets/605102e2-4831-4fc3-ade6-db4a407ee84b" />
<img width="1812" height="699" alt="image" src="https://github.com/user-attachments/assets/a082533e-57f8-4c40-8501-d5cb4b503742" />

