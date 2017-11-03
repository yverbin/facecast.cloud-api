В объектах используются поля Nanoseconds, которые подразумевают указание времени в наносекундах.

Учётные данные:
```
api_url = "https://api.facecast.io/v1/"; — адрес для обращений к API
fc_user = "user"; — логин
fc_key = "key"; — пароль
```

***********************

Заголовки каждого запроса следующие:
```
request.header({
    'Accept': 'application/json',
    'Content-Type': 'application/json',
    'Authorization': 'Basic ' + base64.encode(fc_user + ':' + fc_key).toString('utf-8')
});
```
Заголовок «Authorization» аутентифицирует вас, формируется по приведенной формуле.

***********************
Streamer — «устройство ввода», в вашем случае это браузерный захват. Создаётся для каждой трансляции.
```
streamer = {
    "Type": "WEBRTC_STREAMER"
    "Name": "some_human_readable_name",
    "Key": "unique_streaming_key",
    "Enabled": true,
    "TranscoderID": 2
};
```
Type — тип ввода, у вас это WEBRTC_STREAMER.
Name — уникальное имя устройства ввода, формируется вами.
Key —  ключ в формате [a-zA-Z0-9], используется для аутентификации потока, передаётся софтине браузерного захвата (её предоставляем мы).
Enabled — true/false, можно стримить в это устройство или нет.
TranscoderID — 2 = 720p + 360p + 180p для вашего юзкейса подходит отлично.

***********************
Собственно, сама трансляция.
```
event = {
    "Name": "some_human_readable_name",
    "DVR": {
      "Nanoseconds": 86400000000000
    },
    "Buffer": {
      "Nanoseconds": "3000000000"
    },
    "Expires": {
      "Nanoseconds": 1509822645000
    },
    "Enabled": true
};
```
Name — имя эвента на ваше усмотрение.
DVR — размер окна DVR. В примере — 24 часа.
Buffer — размер предбуфера, пока не накопим в показ не отдаём, нужен чтобы нивелировать проблемы со связью.
Expires — абсолютный таймстамп в наносекундах, когда эвент будет удалён. 

***********************

Элементы расписания, связывают трафик из Streamer и Event. Позволяет собрать итоговый плейлист из кусков
```
schedule = {
    "Schedules": [
        {
            "TimeRange": {
                "from": {
                    "Nanoseconds": 1509711534000
                },
                "to": {
                    "Nanoseconds": 1509822645000
                }
            },
            "StreamerID": StreamerID,
            "EventID": EventID,
            "SequenceNumber": 1,
            "Enabled": true
        }
    ]
};
```
From, To — отметки абсолютного времени в формате UNIX timestamp_nanoseconds откуда и до куда в элемент войдёт трафик.
SequenceNumber — очередность элемента в последовательности.
