# ips-wheel-android13-fix

Патч APK для приложения управления моноколесом (IPS) для запуска на **Android 12/13**, где оригинальные сборки переставали видеть Bluetooth-устройства и вылетали.

## Файлы

- **`ips_wheel_android13.apk`** — исправленная сборка (основа: `ips_20160415.apk`, v1.2 — последняя официальная версия приложения IPS).

## Что было исправлено

### 1. Bluetooth на Android 12/13
Приложение собрано в эпоху `targetSdk 22` и запрашивало только устаревшие права:

```xml
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```

На Android 12+ правила изменились: для BLE-сканирования и подключения нужны
`BLUETOOTH_SCAN` и `BLUETOOTH_CONNECT`. Без них система молча не отдавала результаты
сканирования — приложение работало, но не «видело» устройство.

В манифест добавлено:

```xml
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"
    android:usesPermissionFlags="neverForLocation"/>
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

- `neverForLocation` — сканирование не зависит от геолокации;
- старые права `BLUETOOTH`/`BLUETOOTH_ADMIN` сохранены, чтобы система продолжала
  применять легаси-совместимость к старому коду;
- `targetSdk` остался 22 — в старом коде нет запросов runtime-разрешений, при
  поднятии SDK приложение полностью теряло Bluetooth.

### 2. Диагностика вылетов
Встроена ловушка crash-исключений (`CrashSaver`): при падении показывает
на экране окно «Ошибка приложения» с полным стеком — удобно для отладки на
телефоне без adb.

## Как установить

1. Старую версию приложения удалить (подпись другая — debug-ключ).
2. Установить `ips_wheel_android13.apk` (разрешить установку из неизвестных источников).
3. В **Настройки → Приложения → приложение → Разрешения** включить:
   - «Ближайшие устройства» (обязательно);
   - «Местоположение» (желательно).

## Как собрать из исходников

Декомпилированный проект (smali) не выложен; изменения вносились через
`apktool` (v2.9.3) по стандартному циклу:

```
apktool d orig.apk -o decoded
# правки AndroidManifest.xml + добавление CrashSaver*.smali
apktool b decoded -o fixed-unsigned.apk
uber-apk-signer --apks fixed-unsigned.apk
```

## Дисклеймер

Сборка сделана как хобби-фикс: на основе декомпилированного APK, подписана
debug-ключом, не является официальной версией. Все права на оригинальное
приложение и торговые марки принадлежат их правообладателям. Оригинальный APK:
`ips_20160415.apk` (v1.2, revision от 15.04.2016).

## История проблемы

| Устройство | Симптом до | После фикса |
|---|---|---|
| Samsung S20 FE (Android 13) | не видит моноколесо по BLE; вылет в меню | сканирует и подключается |