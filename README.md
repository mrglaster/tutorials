
## Усложнения 

### Альтернативная атака

Weaponmod поддерживает возможность добавления альтернативной атаки. Для её реализации вам необходимо сделать следующее: 

1) в ```plugin_ini``` вам необходимо зарегистрировать форвард на дополнительную атаку. Выглядеть это будет следующим образом:

```
public plugin_init(){
	...
	wpnmod_register_weapon_forward(iSG, Fwd_Wpn_SecondaryAttack, "Weapon_SecondaryAttack"); // Регистрируем форвард на дополнительную атаку 	
}
```

2) Теперь вам необходимо реализовать натив ```Weapon_SecondaryAttack```. Реализация у него точно такая же, как и у метода для основной атаки.

```
public Weapon_SecondaryAttack(const iItem, const iPlayer) {
    // Ваш код натива
}
```

### Собственный тип боеприпасов

Одной из возможностей Weapon Mod является создание уникальных типов боеприпасов, которые не существуют в оригинальной игре Half-Life. Это позволяет реализовать, например, энергетические патроны, крио-заряды, кислотные гранаты и прочие фантастические боеприпасы, не конфликтуя с уже существующими типами вроде "9mm", "357" или "uranium". Рассмотрим создание собственного типа боеприпасов. 

1) Зададим имена типу патронов и класснейм аммобоксу с ними

```
#define WEAPON_PRIMARY_AMMO		"762" // Это название будет использовано при регистрации оружия
#define AMMOBOX_CLASSNAME		"ammo_762" // Класс для оружейного ящика
#define MODEL_CLIP			"models/w_m40a1clip.mdl" // Модель для подбираемых патронов
```

2) Зарегистрируем новый тип боезапаса. В ```plugin_init``` вам необходимо прописать:

```
public plugin_init(){
    new iAmmo762 = wpnmod_register_ammobox(AMMOBOX_CLASSNAME); // Регистрируем новый тип боезапаса
    wpnmod_register_ammobox_forward(iAmmo762, Fwd_Ammo_Spawn, "Ammo762_Spawn"); // В методе Ammo762_Spawn мы задаем модель для подбираемых поеприпасов
    wpnmod_register_ammobox_forward(iAmmo762, Fwd_Ammo_AddAmmo, "Ammo762_AddAmmo"); // Логика выдачи патронов игроку
}
```

3) Реализуем метод, вызываемый при спавне патронов

```
public Ammo762_Spawn(const iItem)
{
	// Setting world model
	SET_MODEL(iItem, MODEL_CLIP);
}
```

4) Реализуем логику подбора боезапаса

```
//**********************************************
//* Extract ammo from box to player.           *
//**********************************************

public Ammo762_AddAmmo(const iItem, const iPlayer)
{
	new iResult = 
	(
		ExecuteHamB
		(
			Ham_GiveAmmo, 
			iPlayer, 
			WEAPON_MAX_CLIP, 
			WEAPON_PRIMARY_AMMO, 
			WEAPON_PRIMARY_AMMO_MAX
		) != -1
	);
	
	if (iResult)
	{
		emit_sound(iItem, CHAN_ITEM, "items/9mmclip1.wav", 1.0, ATTN_NORM, 0, PITCH_NORM);
	}
	
	return iResult;
}
```

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
