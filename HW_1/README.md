# itis-2026-dataBase-atushkina
Запрос 1: Найти названия и длительность фильмов с возрастным ограничением не выше 16+

π title, time_of_film (σ age_rating = '16+' (film))

Запрос 2: Показать название фильма, номер зала и время сеанса, чтобы увидеть расписание

π film.title, hall.number, session_.date_time ((film ⋈ session_) ⋈ hall)
