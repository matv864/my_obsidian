# Лабораторная работа 6.
## Тестирование гипотезы о значимости различия средних

- выполнил:
студент направления Б9123-01.03.02
Иванов Матвей Олегович

- принял:
Деревягин А.А

---

# задание 1
```
Напишите функцию, проверяющую гипотезу о незначимости различий между средними двух нормальных генеральных совокупностей. Функция по переданным параметрам должна учитывать, что: 1) дисперсии могут быть известны, а могут быть неизвестны (в этом случае внимательно прочитайте условия применения критерия), 2) выборки могут быть зависимыми, а могут быть независимыми, 3) критерий может быть право-, лево-, дву-сторонним. Результат функции — p-значение или ошибка в случае возникновения причин невозможности проведения теста. Результаты собственной функции нужно сверить с результатами встроенного метода.
```

# матчасть
Мы хотим проверить:
$H0:μ1=μ2 \,\, \text{ против } \,\, H1:μ1\ne μ2 \,\,(\text{или}\,\, >, <)$

**Ключевые моменты:**
1. Дисперсии известны → Z-критерий (нормальное распределение).
2. Дисперсии неизвестны → t-критерий (распределение Стьюдента).
3. Выборки зависимые → парный t-критерий (сводим к одной выборке разностей).

### моя реализация
```python
def universal_mean_test(sample1, sample2, alternative="two-sided", std1=None, std2=None, isdependent=False):
if isdependent:
	# t-test для зависимых выборок
	return dependent_t_test(sample1, sample2)
elif std1 and std2:
	# z-test для известных дисперсий (независимые выборки)
	return Z_test(sample1, sample2, std1, std2, alternative)
else:
	if Fischer_test(sample1, sample2) > 0.05:
		return T_test(sample1, sample2, alternative)
	else:
		return Welch_t_test(sample1, sample2, alternative)
```


### через scipy.stats
```python
def universal_mean_test_scipy(sample1, sample2, alternative="two-sided", std1=None, std2=None, isdependent=False, alpha=0.05):
"""Сравнение с использованием scipy.stats"""

if isdependent:
	p_value = stats.ttest_rel(sample1, sample2, alternative=alternative)[1]

elif std1 and std2:
	# z-test для известных дисперсий (независимые выборки)
	n = len(sample1)
	m = len(sample2)
	z_statistic = (np.mean(sample1) - np.mean(sample2)) / (std1**2/n + std2**2/m)**.5
	# Корректный расчет p-value
	match alternative:
		case 'less':
			p_value = stats.norm.cdf(z_statistic)
		case 'greater':
			p_value = 1 - stats.norm.cdf(z_statistic)
		case 'two-sided':
			p_value = 2 * stats.norm.cdf(-np.abs(z_statistic))
else:
	# t-test для неизвестных дисперсий (независимые выборки)
	# Проверка равенства дисперсий (как в universal_mean_test)
	fischer_p_value = Fischer_test(sample1, sample2)
	if fischer_p_value < alpha:
		p_value = stats.ttest_ind(sample1, sample2, alternative=alternative, equal_var=True)[1]
	else:
		p_value = stats.ttest_ind(sample1, sample2, alternative=alternative, equal_var=False)[1]
return p_value
```
# решаем задачи
## задача 1
![[6_task_1.png]]

```python
sample1_z1 = [130] * 30
sample2_z1 = [125] * 40
std1_z1 = 60**(1/2)
std2_z1 = 80**(1/2)

result_z1 = universal_mean_test(sample1_z1, sample2_z1, alternative="two-sided", std1=std1_z1, std2=std2_z1)
result_scipy_z1 = universal_mean_test_scipy(sample1_z1, sample2_z1, alternative="two-sided", std1=std1_z1, std2=std2_z1)
print(f"Результат теста (мой) : {result_z1}")
print(f"Результат теста (scipy): {result_scipy_z1}")
print(f"Вывод: p = {result_scipy_z1:.4f}")
```
## задача 2
![[6_task_2.png]]


```python
sample1_z2 = [3.4]*2+[3.5]*3+[3.7]*4+[3.9]
sample2_z2 = [3.2]*2+[3.4]*2+[3.6]*8

result_z2 = universal_mean_test(sample1_z2, sample2_z2, alternative="two-sided")
result_scipy_z2 = universal_mean_test_scipy(sample1_z2, sample2_z2, alternative="two-sided", alpha=0.05) # Используем alpha=0.05 для проверки дисперсий в scipy-функции
print(f"Результат теста (мой) : {result_z2}")
print(f"Результат теста (scipy): {result_scipy_z2}")
print(f"Вывод: p = {result_scipy_z2:.5f}")
```

## задача 3

![[6_task_3.png]]

```python
sample1_z3 = [2, 3, 5, 6, 8, 10]
sample2_z3 = [10, 3, 6, 1, 7, 4]
result_z3 = universal_mean_test(sample1_z3, sample2_z3, alternative="two-sided", isdependent=True)
result_scipy_z3 = universal_mean_test_scipy(sample1_z3, sample2_z3, alternative="two-sided", isdependent=True)

print(f"Результат теста (мой) : {result_z3}")
print(f"Результат теста (scipy): {result_scipy_z3}")
print(f"Вывод: p = {result_scipy_z3:.3f}")
```

---
ссылка на код: https://github.com/matv864/AI_work/tree/main/matstat
