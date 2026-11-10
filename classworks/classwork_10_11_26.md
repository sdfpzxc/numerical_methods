# 12. Мини-упражнение

1. Создайте массив формы `(6, 4)` из `np.arange(24)`. Выведите `.shape`, `.ndim`, `.size`, `.dtype`.
2. Перестройте его в `(4, 6)`, затем в `(2, 3, 4)`, затем обратно в `(24,)`. Убедитесь через `np.array_equal`, что значения не изменились.
3. Возьмите столбец `[:, 2]` — это view или копия? Проверьте: измените элемент среза и посмотрите на оригинал.
4. Отфильтруйте элементы `> 10` булевой маской. Посчитайте их среднее.
5. Вычтите из каждой строки её среднее (`axis=1`, keepdims=True) — убедитесь, что результат имеет ту же форму и сумма по строкам ≈ 0.

```python
import numpy as np

# 1
arr = np.arange(24).reshape(6, 4)
print("Meow task 1: ", arr.shape, arr.ndim, arr.size, arr.dtype)

# 2
arr2 = arr.reshape(4, 6)
arr3 = arr.reshape(2, 3, 4)
arr4 = arr3.reshape(24)
print("Meow task 2: ",np.array_equal(arr4, np.arange(24)))

# 3
col = arr[:, 2]
col[0] = 999
print("Meow task 3: ",arr[0, 2])

# 4
masked = arr[arr > 10]
print("Meow task 4: ",np.mean(masked))

# 5
arr_centered = arr - np.mean(arr, axis=1, keepdims=True)
print("Meow task 5: ",arr_centered.shape)
print(np.sum(arr_centered, axis=1))
```

```python
Meow task 1:  (6, 4) 2 24 int64
Meow task 2:  True
Meow task 3:  999
Meow task 4:  87.14285714285714
Meow task 5:  (6, 4)
[0. 0. 0. 0. 0. 0.]
PS C:\Users\User\Desktop\hw_2_coursss\n_m>
```
