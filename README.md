# Импортруем библиотеку pandas для анализа табличных данных
import pandas as pd

used_by_user = list()


# Напишем функцию которая бы считывала файлы по указанным путям и выдавала 10 лучших фильмов независимо от количества отзывов
def find_best_movies(ratings_file, movies_file):
    global used_by_user
    # Считываем данные с файлов
    ratings_data = pd.read_csv(ratings_file)
    movies_data = pd.read_csv(movies_file, on_bad_lines='skip')  # Так как некоторые строки в файле Movies некорректны из-за разделителя - пропустим их

    # Переведем значение movieId в int
    ratings_data['movieId'] = ratings_data['movieId'].astype(int)
    movies_data['movieId'] = movies_data['movieId'].astype(int)
    used_by_user = [list() for i in range(len(ratings_data))]
    for index, row in ratings_data.iterrows():
        used_by_user[int(row['userId'])].append(row['movieId'])
    # print(used_by_user)

    # used_by_user[ratings_data['userId']] = ratings_data['movieId']
    # Посчитаем рейтинг фильмов
    average_ratings = ratings_data.groupby('movieId')['rating'].mean().reset_index()

    # Совместим рейтинг с названием фильма через movieId
    combined_data = pd.merge(average_ratings, movies_data, on='movieId')

    # Отсортрируем по убыванию рейтинга
    sorted_movies = combined_data.sort_values(by='rating', ascending=False)

    return sorted_movies


# Указываем путь к файлам
ratings_path = r'ratings1.csv'
movies_path = r'movies.csv'

# Вызовем функцию для наших файлов
best_movies = find_best_movies(ratings_path, movies_path)
print("Введите количество запросов: ")
num_users = int(input())
for i in range(num_users):
    print("Введите номер пользователя и количество фильмов для рекомендации: ")
    user_id, num_films = map(int, input().split())

    cnt = 0;
    for index, row in best_movies.iterrows():
        if cnt == num_films:
            break
        if not row['movieId'] in used_by_user[user_id]:
            print(f"Название (год): {row['title']}, Рейтинг:  {row['rating']}, Жанр: {row['genres']}")
            cnt += 1
