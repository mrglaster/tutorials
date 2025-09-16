# Создание собственного оружия для Haf-Life средствами WeaponMod 

В 2012-м году KORD_12.7 начал написание статьи по разработке собственного оружия с использованием Weapon Mod. Статья так и не была завершена. Исправить это недразумение постараемся мы. В данной статье будут расмотрены методы создания нового оружия для Half-Life средствами модуля Weapon Mod, начиная от простого, заканчивая наворотами.  

## Создание простого оружия 

### Регистрация оружия 

В качестве самого первого примера реализуем винтовку M16, имеющую только автоматический режим огня и использующую патроны от MP5.

Для начала подключим необходимый модуль:

```
#include <amxmodx>
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

Если затронули модели, то опишем уже и все ресурсы, используемые плагином: 

```
// Models
#define MODEL_WORLD	"models/hl-hev/m16a1/w_m16a1ep.mdl"
#define MODEL_VIEW	"models/hl-hev/m16a1/v_m16a1ep_hev.mdl"
#define MODEL_PLAYER	"models/hl-hev/m16a1/p_m16a1ep.mdl"

// Hud
#define WEAPON_HUD_TXT	"sprites/weapon_m16a1ep.txt"
#define WEAPON_HUD_BAR	"sprites/640hud79.spr"

// Sounds
#define SOUND_FIRE	"weapons/hl-hev/m16a1/m16a1ep_shoot.wav"
#define SOUND_BOLTUP 	"weapons/hl-hev/m16a1/m16a1ep_boltpull.wav"
#define SOUND_RELOAD	"weapons/hl-hev/m16a1/m16a1ep_clipout.wav"
#define SOUND_DEPLOY "weapons/hl-hev/m16a1/m16a1ep_deploy.wav"
#define SOUND_CLIPIN "weapons/hl-hev/m16a1/m16a1ep_clipin.wav"
```

И загрузим их в методе ```plugin_precache```

```
public plugin_precache()
{
	PRECACHE_MODEL(MODEL_VIEW);
	PRECACHE_MODEL(MODEL_WORLD);
	PRECACHE_MODEL(MODEL_PLAYER);
	
	PRECACHE_SOUND(SOUND_RELOAD);
	PRECACHE_SOUND(SOUND_FIRE);
	PRECACHE_SOUND(SOUND_CLIPIN);
	PRECACHE_SOUND(SOUND_BOLTUP);
	PRECACHE_SOUND(SOUND_DEPLOY);
	
	PRECACHE_GENERIC(WEAPON_HUD_TXT);
	PRECACHE_GENERIC(WEAPON_HUD_BAR);
}
```



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
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Spawn,"M16_Spawn"); // Спавн оружия на земле
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Deploy,"M16_Deploy"); // Появление оружия (опционально; используй, если есть анимация)
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Holster,"M16_Holster"); // Анимация смены оружия (опционально; используй, если есть анимация)
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Idle,"M16_Idle"); // Поведение в состоянии Idle
wpnmod_register_weapon_forward(M16,Fwd_Wpn_PrimaryAttack,"M16_PrimaryAttack"); // Хук для основной атаки
wpnmod_register_weapon_forward(M16,Fwd_Wpn_Reload,"M16_Reload"); // Хук для перезарадки
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

	wpnmod_register_weapon_forward(M16,Fwd_Wpn_Spawn,"M16_Spawn"); // Спавн оружия на земле
	wpnmod_register_weapon_forward(M16,Fwd_Wpn_Deploy,"M16_Deploy"); // Появление оружия (опционально; используй, если есть анимация)
	wpnmod_register_weapon_forward(M16,Fwd_Wpn_Holster,"M16_Holster"); // Анимация смены оружия (опционально; используй, если есть анимация)
	wpnmod_register_weapon_forward(M16,Fwd_Wpn_Idle,"M16_Idle"); // Поведение в состоянии Idle
	wpnmod_register_weapon_forward(M16,Fwd_Wpn_PrimaryAttack,"M16_PrimaryAttack"); // Хук для основной атаки
	wpnmod_register_weapon_forward(M16,Fwd_Wpn_Reload,"M16_Reload"); // Хук для перезарадки
}
```
### Реализация хуков 

#### Спавн оружия 

Начнем с реализации метода ```M16_Spawn```, который срабатывает при спавне сущности оружия. В нем мы задаем модель оружия, а также количество боеприпасов, которое получит игрок при подборе. 

```
public M16_Spawn(const iItem)
{
	//Set model to floor
	SET_MODEL(iItem, MODEL_WORLD);
	
	// Give a default ammo to weapon
	wpnmod_set_offset_int(iItem, Offset_iDefaultAmmo, WEAPON_DEFAULT_AMMO);
}
```

#### "Развертывание" оружия 

Теперь реализуем метод ```M16_Deploy```. Этот метод вызывается, когда игрок достает данное оружие. В методе мы задаем view модель, анимацию доставания, время, через которое можно начать стрельбу, а также проигрываем звук, если это необходимо.

```
public M16_Deploy(const iItem, const iPlayer, const iClip)
{
	wpnmod_set_offset_float(iItem, Offset_flNextPrimaryAttack, 1.0); // Стрелять сможем начать через 1 секунду
	wpnmod_set_offset_float(iItem, Offset_flTimeWeaponIdle, 1.2); // Если ничего не делаем, то переходим в состояние Idle через 1.2 секунды.
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_DEPLOY, 0.9, ATTN_NORM, 0, PITCH_NORM);
	return wpnmod_default_deploy(iItem, MODEL_VIEW, MODEL_PLAYER, ANIM_DRAW, ANIM_EXTENSION);
}
```
#### Состояние покоя 

Для обработки состояния Idle реализуем соответствующий метод. 

```
public M16_Idle(const iItem)
{
	wpnmod_reset_empty_sound(iItem);

	
	if (wpnmod_get_offset_float(iItem, Offset_flTimeWeaponIdle) > 0.0)
	{
		return;
	}

	wpnmod_send_weapon_anim(iItem, ANIM_IDLE); // Анимация состояния покоя 
	wpnmod_set_offset_float(iItem, Offset_flTimeWeaponIdle, 6.0); // Через сколько анимация будет проиграна повторно
}
```

#### Убираем оружие в инвентарь 

Реализуем метод обратный методу доставания - метод ```M16_Holster```. В методе мы указываем анимацию, которая будет проиграна при смене оружия, а также останавливаем все прочие процессы (например, перезарядку). 

```
public M16_Holster(const iItem , iPlayer)
{
	// Cancel any reload in progress.
	wpnmod_set_offset_int(iItem, Offset_iInSpecialReload, 0);
	// wpnmod_send_weapon_anim(iItem, ANIM_HOLSTER) - В нашей v_ модели нет такой анимации. Поэтому строка закомментирована. 
}
```


#### Перезарядка

Реализуем метод ```M16_Reload```, отвечающий за перезарядку. В Weaponmod за перезарядку отвечает функция ```wpnmod_default_reload``` и одним из её аргументов является длительность анимации перезарядке. Оно рассчитывается как Количество кадров анимации  / FPS анимаци. Узнать их можно в любом просмотрщике моделей, выбрав соответствующую анимацию.  В нашем случае количество кадров было 114, а FPS 30. Получаем значение 3.8.

<img width="457" height="101" alt="image" src="https://github.com/user-attachments/assets/239367c3-9db5-4af3-8b38-549690ac1ad3" />


```
public M16_Reload(const iItem, const iPlayer, const iClip, const iAmmo)
{
	if (iAmmo <= 0 || iClip >= WEAPON_MAX_CLIP)
	{
		return;
	}
	
	wpnmod_default_reload(iItem, WEAPON_MAX_CLIP, ANIM_RELOAD, 3.8); // 3.8 - длительность анимации перезарядки

	// Проигрыш отдельных звуков при перезарядке. Опционально, если проиграть только один звук или не проигрывать звуков вовсе, то ничего не поменяется.
	//Clip out
	set_task(1.8 , "M16_Reload_Step1" , iPlayer);
	//Clip in
	set_task(3.0 , "M16_Reload_Step2" , iPlayer);
	//Boltup
	set_task(3.8 , "M16_Reload_Step1" , iPlayer);
}

public M16_Reload_Step1(iPlayer)
{
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_RELOAD, 0.9, ATTN_NORM, 0, PITCH_NORM);
}

public M16_Reload_Step2(iPlayer)
{
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_CLIPIN, 0.9, ATTN_NORM, 0, PITCH_NORM);
}
public M16_Reload_Step3(iPlayer)
{
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_BOLTUP, 0.9, ATTN_NORM, 0, PITCH_NORM);
}
```

#### Стрельба 

Передем к самому главному - стрельбе. Реализуем метод 

```
public M16_PrimaryAttack(const iItem, const iPlayer, iClip)
{
	// Проверка возможна ли стрельба: если нет патронов в магазине или игрок под водой - не стреляем
	if (pev(iPlayer, pev_waterlevel) == 3 || iClip <= 0)
	{
		wpnmod_play_empty_sound(iItem);
		wpnmod_set_offset_float(iItem, Offset_flNextPrimaryAttack, 0.15);
		return;
	}

	wpnmod_set_offset_int(iItem, Offset_iClip, iClip -= 1); // Уменьшаем количество патронов в обойме на 1
	wpnmod_set_offset_int(iPlayer, Offset_iWeaponVolume, LOUD_GUN_VOLUME); // Настройка громкости выстрела
	wpnmod_set_offset_int(iPlayer, Offset_iWeaponFlash, BRIGHT_GUN_FLASH); // Вспышка от выстрела

	wpnmod_set_offset_float(iItem, Offset_flNextPrimaryAttack, 0.08); // Через сколько можно выстрелить в следующий раз
	wpnmod_set_offset_float(iItem, Offset_flTimeWeaponIdle, 1.0);

	wpnmod_set_player_anim(iPlayer, PLAYER_ATTACK1); // Анимация модели игрока 
	wpnmod_send_weapon_anim(iItem, ANIM_SHOOT1); // Анимация от первого лица

	wpnmod_fire_bullets(
		iPlayer, // Игрок
		iPlayer, // Атакующий
		1, // Количество выпускаемых пуль
		VECTOR_CONE_1DEGREES, // Как будут лететь пули: d 
		8192.0, // Максимальное дистанция выстрела
		WEAPON_DAMAGE, // Урон 1 пули
		DMG_BULLET | DMG_NEVERGIB, // Тип наносимого урона
		1 // как часто пули будут оставлять след в воздухе
	);

	emit_sound(iPlayer, CHAN_WEAPON, SOUND_FIRE, 0.9, ATTN_NORM, 0, PITCH_NORM); // Проигрываем звук выстрела

	set_pev(iPlayer, pev_effects, pev(iPlayer, pev_effects) | EF_MUZZLEFLASH); // Эффект вспышки
	set_pev(iPlayer, pev_punchangle, Float:{-1.0, 0.0, 0.0}); // Отдача
}
```
#### Итог

По итогу мы получаем следующий код простого плагина: 

```
#include <amxmodx>
#include <hl_wpnmod>

#define PLUGIN "Weapon M16"
#define VERSION "1.0"
#define AUTHOR "Demo"

// Основные параметры оружия
#define WEAPON_NAME "weapon_m16a1ep"
#define WEAPON_SLOT 4
#define WEAPON_POSITION 5
#define WEAPON_PRIMARY_AMMO "9mmAR"
#define WEAPON_PRIMARY_AMMO_MAX 200
#define WEAPON_SECONDARY_AMMO ""
#define WEAPON_SECONDARY_AMMO_MAX 0
#define WEAPON_MAX_CLIP 30
#define WEAPON_DEFAULT_AMMO 200
#define WEAPON_FLAGS 0
#define WEAPON_WEIGHT 20
#define WEAPON_DAMAGE 70.0
#define ANIM_EXTENSION "mp5"

// Анимации
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

// Модели
#define MODEL_WORLD "models/hl-hev/m16a1/w_m16a1ep.mdl"
#define MODEL_VIEW "models/hl-hev/m16a1/v_m16a1ep_hev.mdl"
#define MODEL_PLAYER "models/hl-hev/m16a1/p_m16a1ep.mdl"

// HUD
#define WEAPON_HUD_TXT "sprites/weapon_m16a1ep.txt"
#define WEAPON_HUD_BAR "sprites/640hud79.spr"

// Звуки
#define SOUND_FIRE "weapons/hl-hev/m16a1/m16a1ep_shoot.wav"
#define SOUND_BOLTUP "weapons/hl-hev/m16a1/m16a1ep_boltpull.wav"
#define SOUND_RELOAD "weapons/hl-hev/m16a1/m16a1ep_clipout.wav"
#define SOUND_DEPLOY "weapons/hl-hev/m16a1/m16a1ep_deploy.wav"
#define SOUND_CLIPIN "weapons/hl-hev/m16a1/m16a1ep_clipin.wav"

// Глобальный ID оружия
new g_iM16;

public plugin_precache()
{
	PRECACHE_MODEL(MODEL_VIEW);
	PRECACHE_MODEL(MODEL_WORLD);
	PRECACHE_MODEL(MODEL_PLAYER);

	PRECACHE_SOUND(SOUND_RELOAD);
	PRECACHE_SOUND(SOUND_FIRE);
	PRECACHE_SOUND(SOUND_CLIPIN);
	PRECACHE_SOUND(SOUND_BOLTUP);
	PRECACHE_SOUND(SOUND_DEPLOY);

	PRECACHE_GENERIC(WEAPON_HUD_TXT);
	PRECACHE_GENERIC(WEAPON_HUD_BAR);
}

public plugin_init()
{
	register_plugin(PLUGIN, VERSION, AUTHOR);

	g_iM16 = wpnmod_register_weapon(
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

	// Регистрация хуков
	wpnmod_register_weapon_forward(g_iM16, Fwd_Wpn_Spawn, "M16_Spawn");
	wpnmod_register_weapon_forward(g_iM16, Fwd_Wpn_Deploy, "M16_Deploy");
	wpnmod_register_weapon_forward(g_iM16, Fwd_Wpn_Holster, "M16_Holster");
	wpnmod_register_weapon_forward(g_iM16, Fwd_Wpn_Idle, "M16_Idle");
	wpnmod_register_weapon_forward(g_iM16, Fwd_Wpn_PrimaryAttack, "M16_PrimaryAttack");
	wpnmod_register_weapon_forward(g_iM16, Fwd_Wpn_Reload, "M16_Reload");
}

// Спавн оружия на земле
public M16_Spawn(const iItem)
{
	SET_MODEL(iItem, MODEL_WORLD);
	wpnmod_set_offset_int(iItem, Offset_iDefaultAmmo, WEAPON_DEFAULT_AMMO);
}

// Доставание оружия
public M16_Deploy(const iItem, const iPlayer, const iClip)
{
	wpnmod_set_offset_float(iItem, Offset_flNextPrimaryAttack, 1.0);
	wpnmod_set_offset_float(iItem, Offset_flTimeWeaponIdle, 1.2);
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_DEPLOY, 0.9, ATTN_NORM, 0, PITCH_NORM);
    return wpnmod_default_deploy(iItem, MODEL_VIEW, MODEL_PLAYER, ANIM_DRAW, ANIM_EXTENSION);
}

// Состояние покоя
public M16_Idle(const iItem)
{
	wpnmod_reset_empty_sound(iItem);

	
	if (wpnmod_get_offset_float(iItem, Offset_flTimeWeaponIdle) > 0.0)
	{
		return;
	}

	wpnmod_send_weapon_anim(iItem, ANIM_IDLE); // Анимация состояния покоя 
	wpnmod_set_offset_float(iItem, Offset_flTimeWeaponIdle, 6.0); // Через сколько анимация будет проиграна повторно
}

// Убирание оружия
public M16_Holster(const iItem, iPlayer)
{
	wpnmod_set_offset_int(iItem, Offset_iInSpecialReload, 0);
	// wpnmod_send_weapon_anim(iItem, ANIM_HOLSTER); // Если есть анимация
}

// Idle (ничего не делаем, можно оставить пустым)
public M16_Idle(const iItem, const iPlayer)
{
	// Можно добавить анимацию или звук по таймеру, если нужно
}

// Перезарядка
public M16_Reload(const iItem, const iPlayer, const iClip, const iAmmo)
{
	if (iAmmo <= 0 || iClip >= WEAPON_MAX_CLIP)
	{
		return;
	}

	wpnmod_default_reload(iItem, WEAPON_MAX_CLIP, ANIM_RELOAD, 3.8);

	// Пошаговые звуки перезарядки
	set_task(1.8, "M16_Reload_Step1", iPlayer);
	set_task(3.0, "M16_Reload_Step2", iPlayer);
	set_task(3.8, "M16_Reload_Step3", iPlayer);
}

public M16_Reload_Step1(iPlayer)
{
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_RELOAD, 0.9, ATTN_NORM, 0, PITCH_NORM);
}

public M16_Reload_Step2(iPlayer)
{
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_CLIPIN, 0.9, ATTN_NORM, 0, PITCH_NORM);
}

public M16_Reload_Step3(iPlayer)
{
	emit_sound(iPlayer, CHAN_WEAPON, SOUND_BOLTUP, 0.9, ATTN_NORM, 0, PITCH_NORM);
}

// Основная атака (автоматический огонь)
public M16_PrimaryAttack(const iItem, const iPlayer, iClip)
{
	if (pev(iPlayer, pev_waterlevel) == 3 || iClip <= 0)
	{
		wpnmod_play_empty_sound(iItem);
		wpnmod_set_offset_float(iItem, Offset_flNextPrimaryAttack, 0.15);
		return;
	}

	wpnmod_set_offset_int(iItem, Offset_iClip, --iClip);
	wpnmod_set_offset_int(iPlayer, Offset_iWeaponVolume, LOUD_GUN_VOLUME);
	wpnmod_set_offset_int(iPlayer, Offset_iWeaponFlash, BRIGHT_GUN_FLASH);

	wpnmod_set_offset_float(iItem, Offset_flNextPrimaryAttack, 0.08);
	wpnmod_set_offset_float(iItem, Offset_flTimeWeaponIdle, 1.0);

	wpnmod_set_player_anim(iPlayer, PLAYER_ATTACK1);
	wpnmod_send_weapon_anim(iItem, ANIM_SHOOT1);

	wpnmod_fire_bullets(
		iPlayer,
		iPlayer,
		1,
		VECTOR_CONE_1DEGREES,
		8192.0,
		WEAPON_DAMAGE,
		DMG_BULLET | DMG_NEVERGIB,
		1
	);

	emit_sound(iPlayer, CHAN_WEAPON, SOUND_FIRE, 0.9, ATTN_NORM, 0, PITCH_NORM);
	set_pev(iPlayer, pev_effects, pev(iPlayer, pev_effects) | EF_MUZZLEFLASH);
	set_pev(iPlayer, pev_punchangle, Float:{-1.0, 0.0, 0.0});
}
```

В следующих разделах рассмотрим, как можно разнообразить разрабатываемое вооружение.


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
