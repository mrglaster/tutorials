# Создание собственного оружия для Haf-Life средствами WeaponMod 

В 2012-м году KORD_12.7 начал написание статьи по разработке собственного оружия с использованием Weapon Mod. Статья так и не была завершена. Исправить это недразумение постараемся мы. В данной статье будут расмотрены методы создания нового оружия для Half-Life средствами модуля Weapon Mod, начиная от простого, заканчивая наворотами.  

## Создание простого оружия 

### Регистрация оружия 

В качестве самого первого примера реализуем винтовку M16, имеющую только автоматический режим огня и использующую патроны от MP5.

Для начала подключим необходимый модуль:

```
#include <hl_wpnmod> // Weapon Mod
```

Теперь необходимо задать основные параметры нашего оружия 

```
#define WEAPON_NAME "weapon_m16a1ep" // Название оружия
#define WEAPON_SLOT	3 // Слот
#define WEAPON_POSITION	5 // Положение в слоте
#define WEAPON_PRIMARY_AMMO	"9mmAR" // Название боеприпасов
#define WEAPON_PRIMARY_AMMO_MAX	200 // Максимальное количество патронов
#define WEAPON_SECONDARY_AMMO	"" // Название второго типа боезапаса (например, гранаты для подствольника) 
#define WEAPON_SECONDARY_AMMO_MAX	0 // Их же максимальное количество
#define WEAPON_MAX_CLIP	30 // Количество патронов в одной обойме. Поставьте значение -1, если не хотите разделения на обоймы (перезарядки тоже не будет)
#define WEAPON_DEFAULT_AMMO	 200 // Количество патронов при подборе оружия 
#define WEAPON_FLAGS	0
#define WEAPON_WEIGHT	20 // "Вес" оружия. По этому параметру происходит определение, какое оружие "круче" при автоподборе. 
#define WEAPON_DAMAGE	70.0 // Урон оружия при основной атаке
#define ANIM_EXTENSION	"mp5" // Анимация игрока при использовании оружия
```

Опишем анимации. Делается это, чтобы использовать название анимации вместо её номера.   

```
enum _:cz_VUL
{
	ANIM_IDLE,
	ANIM_SHOOT1,
	ANIM_SHOOT2,
	ANIM_RELOAD,
	ANIM_DRAW,
	ANIM_SHOOT3,
	ANIM_SHOOT4
}; 
```

Порядок их должен быть таким же, как и во вьювере. Названия могут отличаться 

<img width="230" height="131" alt="image" src="https://github.com/user-attachments/assets/b20dcebe-0da9-4aad-92f2-2d5b6d5da4a2" />




Теперь нам необходимо зарегестрировать наше оружие в модуле, для этого используем натив ```wpnmod_register_weapon``` в методе ```plugin_init```. И так, давайте сделаем это. Для этого нам понадобятся созданные ранее макросы.

```
public plugin_init() {
	register_plugin(PLUGIN, VERSION, AUTHOR);
	new M16 = wpnmod_register_weapon
	(
		WEAPON_NAME,
		WEAPON_SLOT,
		WEAPON_POSITION,
		WEAPON_PRIMARY_AMMO,
		WEAPON_PRIMARY_AMMO_MAX,
		WEAPON_SECONDARY_AMMO,
		WEAPON_SECONDARY_AMMO_MAX,
		WEAPON_MAX_CLIP,
		WEAPON_FLAGS,
		WEAPON_WEIGHT
	);
}
```
Мы его зарегистрировали, в M16 находится ID нашего оружия. Что с ним делать? С помошью этого ID мы можем регистрировать различные хуки нашего оружия, для этого используем натив ```wpnmod_register_weapon_forward```. 

```
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Spawn,"fw_M16Spawn"); // Спавн оружия на земле
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Deploy,"fw_M16Deploy"); // Появление оружия (опционально; используй, если есть анимация)
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Holster,"fw_M16Holster"); // Анимация смены оружия (опционально; используй, если есть анимация)
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Idle,"fw_M16Idle"); // Поведение в состоянии Idle
wpnmod_register_weapon_forward(M16,Fwd_Wpn_PrimaryAttack,"fw_M16PrimaryAttack"); // Хук для основной атаки
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Reload,"fw_M16Reload"); // Хук для перезарадки
```

Помимо этих существуют другие хуки, например, на работу с дополнительной атакой или использовании при создании собственных типов боеприпасов. Работа с ними будет описано в разделе "Усложнения".

По итогу ```plugin_init``` принимает следующий вид: 

```
public plugin_init() {
	register_plugin(PLUGIN, VERSION, AUTHOR);
	new M16 = wpnmod_register_weapon
	(
		WEAPON_NAME,
		WEAPON_SLOT,
		WEAPON_POSITION,
		WEAPON_PRIMARY_AMMO,
		WEAPON_PRIMARY_AMMO_MAX,
		WEAPON_SECONDARY_AMMO,
		WEAPON_SECONDARY_AMMO_MAX,
		WEAPON_MAX_CLIP,
		WEAPON_FLAGS,
		WEAPON_WEIGHT
	);

  wpnmod_register_weapon_forward(M16,Fwd_Wpn_Spawn,"fw_M16Spawn"); // Спавн оружия на земле
  wpnmod_register_weapon_forward(M16,Fwd_Wpn_Deploy,"fw_M16Deploy"); // Появление оружия (опционально; используй, если есть анимация)
  wpnmod_register_weapon_forward(M16,Fwd_Wpn_Holster,"fw_M16Holster"); // Анимация смены оружия (опционально; используй, если есть анимация)
  wpnmod_register_weapon_forward(M16,Fwd_Wpn_Idle,"fw_M16Idle"); // Поведение в состоянии Idle
  wpnmod_register_weapon_forward(M16,Fwd_Wpn_PrimaryAttack,"fw_M16PrimaryAttack"); // Хук для основной атаки
  wpnmod_register_weapon_forward(M16,Fwd_Wpn_Reload,"fw_M16Reload"); // Хук для перезарадки

}
```

## Усложнения 


### Альтернативная атака

### Прицеливание 

#### Реализация через спрайт 

#### Реализация через анимацию V_ модели

#### Реалазация через отдельную модель 

### Запуск снарядов 

### Лазерное оружие 

### Создание оружия ближнего боя

## Бонус: полезные методы при работе с оружием 

### Управляемые снаряды 

### Телепортация на альтернативную атаку 

### Повышение скорострельности, если рядом находится тиммейт 

### 
