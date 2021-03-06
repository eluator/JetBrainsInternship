Решались две задачи MountainCar-V0 и MountainCarContinuous-v0. В ноутбуке MountainCar-v0.ipynb решение первой, в ActorCritic.ipynb - второй.

Для MountainCar-V0 использовался DQN алгоритм, основанный на Q-функции. Его преимущество по сравнению с Policy gradient в том, что можно использовать старые данные для обучения. 
1) Создаётся класс Policy на основе модуля torch.nn с двумя линейными слоями(без нелинейности между ними, хотя можно и добавить). На выходе вероятности для каждого их трёх действий.
2) ReplayMemory сохраняет данные в виде Transition и перезаписывает, если объём памяти превышен.
3) Создаётся две нейросети policy_net и target_net. Первая будет обновляться на каждом шаге, вторая через TARGET_UPDATE итераций. target_net используется в расчёте функции потерь для Q-функции.
4) Действие выбирается либо исходя из максимума вероятности, выданной нейросетью, либо случайно. Вероятность случайного выбора уменьшается при увеличении числа итераций.
5) optimize_model() берёт batch данных и осуществляет обратный проход. Функция потерь считается, как MSE между выходом нейросети(текущей версий Q-функции) и ожидаемым максимальным reward'ом, рассчитываемым с помощью target_net(старой версией Q-функции). При чём используются результаты Q-функции для всех возможных действий и соответственно всех возможных следующих состояний. Это возможно, потому что их всего три.
6) В эпизодах запись всех трёх возможных следующих состояний осуществляется с помощью перезаписи состояний среды.
7)Результат - 100% на 50 шагах. Если запускать большее число раз, то ошибки всё же будут.

Для MountainCarContinuous-v0 обычный Policy gradient, где выход нейросети - параметры нормального распределения. За базу взят код задачи CartPole-v0 из dlcourse. Эта задача несколько сложнее, т.к. случайно машинка на горку не забирается и если практически не менять алгоритм, она просто будет стремиться к минимуму действий и нулевому reward. При этом случайные действия в PG делать нельзя - loss функция должна содержать action и state сгенерированный из распределения, задаваемого параметрами нейросети. Зато можно использовать reward shaping.

1. Первые 1000 эпизодов награждаю модельку за победу(100 очков) и на последнем шаге прибавляю к reward разницу между максимальным state[0] за все эпизоды и максимальным state[0] на данной траектории.
2. Следующие 8000 эпизодов добавляю к reward ускорение.
3. Получается моделька, которая всегда заезжает на горку и даже получает положительный reward, но небольшой. Теперь можно её обучить с обычным reward'ом ещё 4000 эпизодов.
4. К концу модель имеет средний reward чуть больше 90, всё ещё всегда заезжает на горку(за 4000 эпизодов обучения ни разу не проиграла).
