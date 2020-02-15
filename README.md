# rust-traffic-daemon

Данное ПО обрабатывает пакеты данных по траффику, преобразовывая их в формат данных для clickhouse.

Пакеты данных приходят в двух версиях.

Версия 1.
```json
{
	"time": "2020-02-15 12:48:49+03",
	"object_id": 101520,
	"data": {
		"tr_cars": [
			{
				"value": 400,
				"lane_num": 0
			},
			{
				"value": 500,
				"lane_num": 1
			}
			{
				"value": 600,
				"lane_num": 2
			}
			{
				"value": 700,
				"lane_num": 3
			}
		]
	}
}
```

Версия 2.
```json
{
	"time": "2020-02-15 12:48:49+03",
	"object_id": 101520,
	"data": {
		"tr_cars": [
			{
				"value": 400,
				"direction": "forward",
				"num": 1
			},
			{
				"value": 500,
				"direction": "forward",
				"num": 2
			},
			{
				"value": 600,
				"direction": "backward",
				"num": 1
			},
			{
				"value": 700,
				"direction": "backward",
				"num": 2
			}
		]
	}
}
```

Данные пакеты необходимо преобразовать в докумет вида:
```json
[
	{
		"time": "2020-02-15 09:48:49",
		"station_id": 101520,
		"direction": "forward",
		"lane": 1,
		"tr_cars": 400
	},
	{
		"time": "2020-02-15 09:48:49",
		"station_id": 101520,
		"direction": "forward",
		"lane": 2,
		"tr_cars": 500
	},
	{
		"time": "2020-02-15 09:48:49",
		"station_id": 101520,
		"direction": "backward",
		"lane": 1,
		"tr_cars": 600
	},
	{
		"time": "2020-02-15 09:48:49",
		"station_id": 101520,
		"direction": "backward",
		"lane": 2,
		"tr_cars": 700
	}
]
```

Для преобразования версии 1 в версию 2 необходимо использовать дополнительный источник данных, вида:
```json
{
	"101520": [
		{
			"lane_num": 0,
			"direction": "forward",
			"num": 1
		},
		{
			"lane_num": 1,
			"direction": "forward",
			"num": 2
		},
		{
			"lane_num": 2,
			"direction": "backward",
			"num": 1
		},
		{
			"lane_num": 3,
			"direction": "backward",
			"num": 2
		}
	]
}
```

Для получения заданий используется очередь в rabbitmq.
Воркер получает object_id + time, выбирает по ним пакет данных одной из двух версий.
Выполняет преобразование к формату для Clickhouse и отправляет данные в нее.

В случае, если пришел пакет версии 1, необходимо либо использовать схему преобразования, находящуюся в памяти, либо загрузить ее из postgres.

Если схему не удалось загрузить, необходимо сохранить в лог ошибку, продолжив обрабатывать задачи дальше.