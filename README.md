# 3sem_lab6
## Homework
### Задание 1
Зарегистрируйтесь на сайте https://openweathermap.org/ для получения ключа (API key) к API от сервиса погоды.
Создайте структуру Weather, содержащую свойства Country(страна), Name(город или название местности), Temp(температура воздуха), Description(описание погоды).
Используя API, получите не менее 50 значений текущей погоды в разных точках мира.
Используйте запрос вида: 
https://api.openweathermap.org/data/2.5/weather?lat={Широта}&lon={Долгота}&appid={API key}
, где:
Широта - дробная величина в диапазоне от -90 до 90. 
Долгота - дробная величина в диапазоне от -180 до 180.
API key - ключ, полученный при регистрации на сайте https://openweathermap.org/.
Значения Широты и Долготы изменяйте случайным образом в заданных диапазонах, если для полученной координаты нет значения Country или Name, следует сгенерировать новые координаты.
На основе полученных данных создайте и заполните коллекцию структур Weather.
С помощью LINQ запросов к созданной коллекции, получите и выведите на консоль следующие данные:

Страну с максимальной и минимальной температурой.
Среднюю температуру в мире.
Количество стран в коллекции.
Первую найденную страну и название местности, в которых Description принимает значение: "clear sky","rain","few clouds"
```
using System.Collections.Generic;
using System.Net.Http;
using System;
using System.Text.Json;
using System.Threading.Tasks;
using System.Linq;





class Program
{
    static async Task Main(string[] args)
    {
        for (int j = 0; j < 10; j++)
        {
            Console.WriteLine();

            string apiKey = "eab787e4b774ddb90e6b0a1939d09d53";
            string apiUrl = "https://api.openweathermap.org/data/2.5/weather";
            //содержат ключ API и URL для обращения к сервису погоды.
            List<Weather> weatherData = new List<Weather>();//Создание списка `weatherData` для хранения объектов `Weather`
            Random random = new Random();

            for (int i = 0; i < 50; i++)
            {
                Console.Write($"{i} - ");//Вывод на консоль номера итерации
                double shirota = random.NextDouble() * (90 +90) + (-90);//Генерация случайных значений для широты и долготы
                double dolgota = random.NextDouble() * (180 +180) + (-180);

                Weather weather = await GetWeatherData(apiUrl, apiKey, shirota, dolgota);//Вызов асинхронного метода `GetWeatherData` для получения данных о погоде.
                if (weather != null)
                {//Проверка полученных данных о погоде и добавление их в список `weatherData
                    weatherData.Add(weather);
                }
            }
            
            var maxTempCountry = weatherData.OrderByDescending(w => w.Temp).FirstOrDefault();//Получение страны с максимальной температурой
            /*OrderByDescending(w => w.Temp) сортирует коллекцию weatherData в порядке убывания значения свойства 
             * Temp каждого объекта. Здесь символ w является аргументом лямбда-выражения и представляет каждый элемент 
             * коллекции по очереди, а w.Temp обращается к свойству Temp этого элемента*/
            var minTempCountry = weatherData.OrderBy(w => w.Temp).FirstOrDefault();
            var averageTemp = weatherData.Average(w => w.Temp);//Получение средней температуры
            var distinctCountriesCount = weatherData.Select(w => w.Country).Distinct().Count();//Получение количества уникальных стран

            var allCountries = weatherData.Select(w => w.Country).Distinct().ToList();//Получение списка всех стран

            var firstClearSky = weatherData.FirstOrDefault(w => w.Description == "clear sky");//Получение первой страны с ясным небом
            var firstRain = weatherData.FirstOrDefault(w => w.Description == "rain");//Получение первой страны с дождем
            var firstFewClouds = weatherData.FirstOrDefault(w => w.Description == "few clouds");//Получение первой страны с небольшими облаками

            Console.WriteLine($"\n\nСтрана с максимальной температурой: {maxTempCountry?.Country}, Температура: {maxTempCountry?.Temp}°C");
            Console.WriteLine($"Страна с минимальной температурой: {minTempCountry?.Country}, Температура: {minTempCountry?.Temp}°C");
            Console.WriteLine($"\nСредняя температура в мире: {averageTemp}°C");
            Console.WriteLine($"Количество стран в коллекции: {distinctCountriesCount}");

            if (firstClearSky != null && firstClearSky.Country != null)
            {
                Console.WriteLine($"\nПервая страна с ясным небом: {firstClearSky.Country}, Местность: {firstClearSky.Name}");
            }
            else
            {
                Console.WriteLine("\nСтран с ясным небом нет");
            }

            if (firstRain != null && firstRain.Country != null)
            {
                Console.WriteLine($"Первая страна с дождем: {firstRain.Country}, Местность: {firstRain.Name}");
            }
            else
            {
                Console.WriteLine("Стран с дождем нет");
            }

            if (firstFewClouds != null && firstFewClouds.Country != null)
            {
                Console.WriteLine($"Первая страна с небольшими облаками: {firstFewClouds.Country}, Местность: {firstFewClouds.Name}");
            }
            else
            {
                Console.WriteLine("Стран с облаками нет");
            }

            Console.WriteLine("\nВсе страны:");
            foreach (var country in allCountries)
            {
                Console.WriteLine(country);
            }
        }
    }

    static async Task<Weather> GetWeatherData(string apiUrl, string apiKey, double shirota, double dolgota)
    {
        using (HttpClient client = new HttpClient())//Создание объекта `HttpClient` для отправки HTTP-запросов
        {
            int maxAttempts = 10;
            int attempt = 0;

            while (attempt < maxAttempts)
            {//Цикл `while` - выполняется, пока количество попыток меньше максимального значения.
                var response = await client.GetStringAsync($"{apiUrl}?lat={shirota}&lon={dolgota}&appid={apiKey}&units=metric");
                //Отправка GET-запроса с использованием `HttpClient` и получение ответа в формате строки
                var weatherInfo = JsonSerializer.Deserialize<WeatherInfo>(response);
                //Десериализация JSON-строки в объект `WeatherInfo`
                if (weatherInfo != null && !string.IsNullOrEmpty(weatherInfo.sys.country))
                {//Проверка, что объект `WeatherInfo` не равен `null` и страна не является пустой строкой
                    string country = weatherInfo.sys.country;
                    string name = weatherInfo.name;
                    double temp = weatherInfo.main.temp;
                    string description = weatherInfo.weather[0].description;

                    return new Weather//Создание и возвращение объекта `Weather` с полученными данными о погоде
                    {
                        Country = country,
                        Name = name,
                        Temp = temp,
                        Description = description
                    };
                }

                attempt++;
            }

            return null;//Возврат `null`, если количество попыток выполнения запроса достигло максимального значения
        }
    }
}

class WeatherInfo
{
    public MainInfo main { get; set; }
    public WeatherDescription[] weather { get; set; }
    public string name { get; set; }
    public SysInfo sys { get; set; }
}

class MainInfo
{
    public double temp { get; set; }
}

class WeatherDescription
{
    public string description { get; set; }
}

class SysInfo
{
    public string country { get; set; }
}

class Weather
{
    public string Country { get; set; }
    public string Name { get; set; }
    public double Temp { get; set; }
    public string Description { get; set; }
}
```
