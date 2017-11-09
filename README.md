**Элементы API**

Посредством API предоставляются стандартные методы CRUD над следующими компонентами:

   - **User**: - изменение свойств пользователя, получение токенов
   - **Streamer**: создание, изменение и удаление устройств и методов ввода медиаданных в платформу
   - **Event**:  создание, изменение и удаление плейлистов и свойств их воспроизведения
   - **Schedules**:  создание, изменение и удаление расписаний, описывающих границы и последователность медиаданных, которые будут представлены в плейлисте
    а так же варианты их преобразования в различные качества
   - **Links**: - получение ссылок с различными свойствами на плейлисты

***********************
**Авторизация**

Авторизация доступа к API  выполняется посредством токенов, передаваемых в качестве паролей. Логином всегда является UserID, получаемый при регистрации в сервисе.
Пароль, полученный при регистрации, так же является токеном, но может быть использован только в методе генерации токенов. Передача идентификационных данных
выполняется стандартной схемой аутентификации http-запросов.

**получение токена: GET  /v1/account/token?expires={duration}**

   - expires - время жизни токена в формате {число}{единица}, например 5s
   - Header - authorization: Basic base64string, где base64string закодированная в base64 строка вида login:token

***********************
**Streamer**

Это  «устройство ввода», например приложение захвата видео из браузера.

***Создание: POST /v1/streamers***

JSON-документ, описывающий Streamer
```
    {
        "Type": "UNKNOWN_STREAMER"
        "Name": "string",
        "Key": "string",
        "Source": "string"
        "Enabled": true
    }
```
где

- Type - тип ввода, значение из списка MEDIABOX_STREAMER, STREAMBOX_STREAMER, RTMP_STREAMER, WEBRTC_STREAMER

            - MEDIABOX_STREAMER - ввод из файлов, сохраненных в облаке MediaBox
            - STREAMBOX_STREAMER - ввод с помощью FACECAST StreamBox
            - RTMP_STREAMER - прием медиаданных посредством RTMP
            - WEBRTC_STREAMER  - прием через приложение захвата видео из браузера

- Name - человекочитаемое имя
- Key - ключ доступа для устройства ввода
- Source - url источника в случае Type=MEDIABOX_STREAMER
- Enabled - флаг включен/выключен

***Результат***
```
    {
         "StreamerID": "string"
         "Type": "UNKNOWN_STREAMER"
         "Name": "string",
         "Key": "string",
         "Source": "string"
         "Enabled": true
    }
```


***Получение: GET /v1/streamers/{StreamerID}***
***Изменение: PUT /v1/streamers/{StreamerID}***
***Удаление: DELETE /v1/streamers/{StreamerID}***

***Результат***
Все методы, кроме удаления возвращают полное описание Streamer, как это происходит при создании. При удалении
возвращается пустой объект

***********************
**Event**

Это плейлист и все его свойства

***Создание: POST /v1/events***

JSON-документ, описывающий Event
```

    {
        "Name": "string",
        "DVR": {
            "Nanoseconds": "string"
        },
        "Buffer": {
            "Nanoseconds": "string"
        },
        "Expires": {
            "Nanoseconds": "string"
        },
       "Enabled": true,
  }
```

где

- Name - человекочитаемое имя
- DVR - размер окна для перемотки в плеере, 0 - доступен весь плейлист
- Buffer - размер буфера воспроизведения. Позволяет за счет отложенного на это значение воспроизведения
обеспечивать непрерывное воспроизведение живой трансляции в случае неустойчивого канала или неустойчивой работы стримера
- Expires - время в utc, после которого плелист будет недоступен
- Enabled - флаг включен/выключен

***Результат***
```
   {
        "EventID": "string"
        "Name": "string",
        "DVR": {
            "Nanoseconds": "string"
        },
        "Buffer": {
            "Nanoseconds": "string"
        },
        "Expires": {
            "Nanoseconds": "string"
        },
       "Enabled": true
  }
```
***Получение: GET /v1/events/{EventID}***
***Изменение: PUT /v1/events/{EventID}***
***Удаление: DELETE /v1/events/{EventID}***

***Результат***
Все методы, кроме удаления возвращают полное описание Event, как это происходит при создании. При удалении
возвращается пустой объект

***********************

***Schedules***

Это элементы, описывающие содержимое плейстов, вид медиаданных и способы их хранения посредством временных интервалов

***Создание: POST /v1/schedules***


JSON-документ, описывающий Schedules
```
{
  "Schedules": [
    {
      "TimeRange": {
        "from": {
          "Nanoseconds": "string"
        },
        "to": {
          "Nanoseconds": "string"
        }
      },
      "StreamerID": "string",
      "EventID": "string",
      "SequenceNumber": 0,
      "Service": {
        "TranscoderID": "string",
        "PrivateReplicaCount": 0,
        "ReplicaCount": 0,
        "DataTTL": {
          "Nanoseconds": "string"
        }
      },
      "Enabled": true,
    }
  ]
}
```

где

- TimeRange - объект, описывающий границы фрагмента медиаданных с указанием начала и конца в наносекундах UTC
- StreamerID - идентификатор Streamer, к медиаданным которого относится TimeRange
- EventID - идентификатор Event, плейлист в котором будет показан данный фрагмент
- SequenceNumber - очередность в плейлисте в котором будет показан данный фрагмент
- Service - объект, описывающий класс обслуживания медиаданных, соответсвующих TimeRange и TimeRange
    -  TranscoderID - идентификатор профиля транскодинга
    -  PrivateReplicaCount - количество реплик на серверах клиента
    -  ReplicaCount - количество реплик в облаке
    -  DataTTL - время жизни медиаданных
- Enabled - флаг включен/выключен

***Получение: GET /v1/schedules?streamer={StreamerID}&event={EventID}&SequenceNumber={SequenceNumber}***
***Изменение: PUT /v1/schedules/{StreamerID}/{EventID}/{SequenceNumber}***
***Удаление: DELETE /v1/schedules?streamer={StreamerID}&event={EventID}&SequenceNumber={SequenceNumber}***

***Результат***
Все методы, кроме удаления возвращают массив Schedules, как это происходит при создании. При удалении
возвращается пустой объект. Get-параметры являются необязательными для получения и удаления, позволяя
выполнять групповые опреации

***********************

***Links***

Эта компонента предназначена для генерации ссылок на плейлисты на стороне сервера. Логика генерации может быть реализована на стороне клиента,
для этого необходимо получить ключ для подписи ссылок и описание алгоритма формирования ссылки.

***Получение ссылки: GET /v1/links/{EventID}?activationttl={duration}&uid={UID}&unique=bool***

где

- EventID -  идентификатор Event, для которого генерируется ссылка
- activationttl - время, в течение которого необходимо активировать ссылку в формате {число}{единица}, например 5s
- uid - поле, используемое клиентом для идентификации зрителя
- unique - флаг, означающий требование обеспечивать невозможность просмотра одновременно с двух или более
устройств с одинаковым uid

***********************


**приложение захвата видео из браузера:**

приложение расположено по адресу https://facecast.net/webcapture/ управление модулем реализовано через GET параметры:

- **gateway** - адрес шлюза (wss://wsgateway.facecast.io)
- **streamerID** - id cтримера, созданного чрезе API
- **key** - ключ стримера

например:  https://facecast.net/webcapture/?gateway=wss://wsgateway.facecast.io&streamerID=fakestreamerid&key=fakestreamerkey

после запуска приложения необходимо согласиться на доступ к камере, микрофону - возможны несколько запросов, потом выбрать камеру и источник звука и нажать Go Live