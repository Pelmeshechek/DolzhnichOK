# ДолжничОК

Учебный прототип сервиса для записи студентов на пересдачи. Проект использует стек из согласованной схемы: .NET 10, ASP.NET Core 10, Blazor Interactive Server, PostgreSQL 17, EF Core 10, Npgsql и ASP.NET Core Identity.

## Состояние проекта

- В Visual Studio откройте `DolzhnikOk.slnx`.
- В проекте есть EF Core-модель, таблицы Identity и начальная миграция PostgreSQL.
- Интерфейсы `/`, `/student` и `/prepodavatel` пока остаются демонстрационными. Вход не выполняет проверку пароля, а запись студента и создание пересдачи пока не сохраняются в БД.
- Identity и роли зарегистрированы в приложении, но страницы входа ещё не связаны с ними и маршруты кабинетов пока не защищены.
- Проект xUnit пока не добавлен.

## Первый запуск в Visual Studio 2026

Нужны .NET 10 SDK и Docker Desktop с включённым Linux-контейнерным движком.

1. Скопируйте `.env.example` в `.env` в корне решения. В Visual Studio можно открыть **Вид → Терминал** и выполнить `Copy-Item .env.example .env`.
2. В том же терминале запустите базу: `docker compose up -d db`.
3. Для удобства локального запуска настройки уже используют те же демонстрационные значения, что и `.env.example`. Если меняете пароль, в **Обозревателе решений** нажмите правой кнопкой на проект `DolzhnikOk`, выберите **Управление секретами пользователя** и обновите строку подключения:

   ```json
   {
     "ConnectionStrings": {
       "DefaultConnection": "Host=localhost;Port=5432;Database=dolzhnikok;Username=dolzhnikok;Password=dolzhnikok_dev_only"
     }
   }
   ```

   Файл секретов пользователя хранится вне репозитория. Если измените пароль в `.env`, поменяйте его и в строке подключения.
4. Откройте **Сервис → Диспетчер пакетов NuGet → Консоль диспетчера пакетов**. Убедитесь, что в поле **Проект по умолчанию** выбрано `DolzhnikOk`, затем выполните `Update-Database`.
5. Запустите проект клавишей **F5** или **Ctrl+F5**.

Для локальной разработки `.env.example` использует пароль `dolzhnikok_dev_only`. Не используйте его для реальных данных или при публикации приложения.

Чтобы остановить контейнер, в терминале Visual Studio выполните `docker compose down`. Данные БД останутся в Docker volume. Команда `docker compose down -v` удаляет этот volume вместе с данными.

## Структура данных

- `Models/ApplicationUser.cs` — пользователь Identity с целочисленным ключом и флагом активности.
- `Models/StudentProfile.cs`, `TeacherProfile.cs`, `StudentGroup.cs`, `Department.cs`, `Subject.cs`, `Classroom.cs` — профили и справочники.
- `Models/ResitSession.cs`, `Booking.cs`, `Comment.cs`, `Notification.cs` — пересдачи и связанные с ними данные.
- `Data/AppDbContext.cs` — связи, уникальные индексы и ограничения целостности.
- `Migrations/` — начальная миграция схемы БД.
- `docker-compose.yml` — PostgreSQL 17 с постоянным Docker volume.

Активные записи студента на пересдачу защищены уникальным индексом. Свободное число мест следует вычислять по активным записям; проверку вместимости нужно выполнять на сервере в транзакции, когда экран записи будет подключён к БД.

## Страницы макета

- `Components/Pages/Home.razor` — экран входа-макет.
- `Components/Pages/Student.razor` — список примерных пересдач и демонстрационная запись.
- `Components/Pages/Teacher.razor` — форма-макет создания пересдачи.
- `Program.cs` — регистрация Blazor, PostgreSQL, Identity и middleware авторизации.
