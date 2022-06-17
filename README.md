# LeoECS Lite Threads - Поддержка многопоточной обработки
Поддержка обработки сущностей в несколько системных потоков.

> Проверено на Unity 2020.3 (не зависит от Unity) и содержит asmdef-описания для компиляции в виде отдельных сборок и уменьшения времени рекомпиляции основного проекта.

# Содержание
* [Социальные ресурсы](#Социальные-ресурсы)
* [Установка](#Установка)
    * [В виде unity модуля](#В-виде-unity-модуля)
    * [В виде исходников](#В-виде-исходников)
* [Пример использования](#Пример-использования)
    * [Component](#Component)
    * [ThreadSystem](#ThreadSystem)
    * [Thread](#Thread)
* [Лицензия](#Лицензия)

# Социальные ресурсы
[![discord](https://img.shields.io/discord/404358247621853185.svg?label=enter%20to%20discord%20server&style=for-the-badge&logo=discord)](https://discord.gg/5GZVde6)

# Установка

## В виде unity модуля
Поддерживается установка в виде unity-модуля через git-ссылку в PackageManager или прямое редактирование `Packages/manifest.json`:
```
"com.leopotam.ecslite.threads": "https://github.com/Leopotam/ecslite-threads.git",
```
По умолчанию используется последняя релизная версия. Если требуется версия "в разработке" с актуальными изменениями - следует переключиться на ветку `develop`:
```
"com.leopotam.ecslite.threads": "https://github.com/Leopotam/ecslite-threads.git#develop",
```

## В виде исходников
Код так же может быть склонирован или получен в виде архива со страницы релизов.

# Пример использования

## Component
```c#
struct C1 {
    public int Id;
}
```
## ThreadSystem
```c#
class TestThreadSystem : EcsThreadSystem<TestThread, C1> {
    protected override int GetChunkSize (EcsSystems systems) {
        // Минимальное количество сущностей, после которого
        // произойдет разделение обработки на новый поток.
        // Этот метод вызывается каждый цикл обновления.
        return 1000;
    }

    protected override EcsWorld GetWorld (EcsSystems systems) {
        // Мир, в котором содержатся фильтры и компонентные пулы.
        return systems.GetWorld ();
    }
    
    protected override EcsFilter GetFilter (EcsWorld world) {
        // Фильтр, сущности из которого будут обрабатываться.
        return world.Filter<C1> ().End ();
    }

    // Дополнительная (опциональная) инициализация данных,
    // которые необходимы для расчетов в потоках.
    protected override void SetData (EcsSystems systems, ref TestThread thread) {
        thread.DeltaTime = Time.deltaTime;
    }
}
```
> **ВАЖНО!** EcsThreadSystem поддерживает до 4 типов компонентов.
 
## Thread
```c#
struct TestThread : IEcsThread<C1> {
    public float DeltaTime;
    int[] _entities;
    C1[] _pool1;
    int[] _indices1;

    public void Init (int[] entities, C1[] pool1, int[] indices1) {
        // Сохранение массива сущностей.
        _entities = entities;
        // Сохранение dense-массива пула компонентов.
        _pool1 = pool1;
        // Сохранение sparse-массива пула компонентов.
        _indices1 = indices1;
    }

    public void Execute (int fromIndex, int beforeIndex) {
        for (int i = fromIndex; i < beforeIndex; i++) {
            var e = _entities[i];
            ref var c1 = ref _pool1[_indices1[e]];
            c1.Id = (c1.Id + 1) % 10000;
        }
    }
}
```
> **ВАЖНО!** Внутри обработчика **запрещено** изменять состояние мира: нельзя создавать / удалять сущности, нельзя добавлять / удалять компоненты на сущности. Допускается только модификация данных внутри существующих компонентов.

# Лицензия
Фреймворк выпускается под двумя лицензиями, [подробности тут](./LICENSE.md).

В случаях лицензирования по условиям MIT-Red не стоит расчитывать на
персональные консультации или какие-либо гарантии.