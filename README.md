# Monitoring

## Установка Prometheus, Node Exporter, Grafana

В данном руководстве установка будет производится вручную. Также этот стек мониторинга можно установить и заупстить с помощью Docker или Ansible. Но для большего понимания установим все вручную для большего понимания.

### Prometheus

Скачиваем с [официального сайта](https://prometheus.io/download/) архив с файлами *prometheus* для своей системы и распаковываем его.

	$ wget https://github.com/prometheus/prometheus/releases/download/v2.22.0/prometheus-2.22.0.linux-amd64.tar.gz
	$ tar xf prometheus-2.22.0.linux-amd64.tar.gz

Создаем необоходимые каталоги и копируем в них необходимые нам файлы.

	# cd prometheus-2.22.0.linux-amd64
	# cp prometheus promtool /usr/local/bin
	# mkdir /etc/prometheus /var/lib/prometheus
	# cp prometheus.yml /etc/prometheus

Создаем пользователя для *prometheus* и назначем его владельцем каталога

	# useradd --no-create-home --home-dir / --shell /bin/false prometheus
	# chown -R prometheus:prometheus /var/lib/prometheus

Создаём systemd-юнит *prometheus.service*

	[Unit]
	Description=Prometheus
	Wants=network-online.target
	After=network-online.target

	[Service]
	User=prometheus
	Group=prometheus
	Restart=on-failure
	ExecStart=/usr/local/bin/prometheus \
	  --config.file /etc/prometheus/prometheus.yml \
	  --storage.tsdb.path /var/lib/prometheus/
	ExecReload=/bin/kill -HUP $MAINPID
	ProtectHome=true
	ProtectSystem=full

	[Install]
	WantedBy=default.target

Перезапускаем *systemd*, запускаем *prometheus* и проверяем его статус

	# systemctl daemon-reload
	# systemctl start prometheus
	# systemctl status prometheus
	● prometheus.service - Prometheus
	   Loaded: loaded (/etc/systemd/system/prometheus.service; disabled; vendor preset: disabled)
	   Active: active (running) since Tue 2020-10-27 13:48:48 UTC; 46min ago
	  Process: 4550 ExecReload=/bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
	 Main PID: 3994 (prometheus)
	   CGroup: /system.slice/prometheus.service
	           └─3994 /usr/local/bin/prometheus --config.file /etc/prometheus/prometheu...
	
Заходим на *http://localhost:9090* и проверяем как работает *prometheus*


### Node Exporter

Как и *prometheus*, аналогично устанавливается *node exporter*, с той лишь разницей что его нужно установить на машины, которые мы будем мониторить. [Официальный сайт с релизами](https://github.com/prometheus/node_exporter/releases).



	$ wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
	$ tar xf node_exporter-1.0.1.linux-amd64.tar.gz

Аналогично *prometheus* создаем необходимые каталоги, копируем туда файлы и создаем пользователя.

	# cd node_exporter-1.0.1.linux-amd64
	# cp node_exporter /usr/local/bin
	# useradd --no-create-home --home-dir / --shell /bin/false node_exporter

Создаём systemd-юнит *node_exporter.service*

	[Unit]
	Description=Prometheus Node Exporter
	After=network.target

	[Service]
	Type=simple
	User=node_exporter
	Group=node_exporter
	ExecStart=/usr/local/bin/node_exporter

	SyslogIdentifier=node_exporter
	Restart=always

	PrivateTmp=yes
	ProtectHome=yes
	NoNewPrivileges=yes

	ProtectSystem=strict
	ProtectControlGroups=true
	ProtectKernelModules=true
	ProtectKernelTunables=yes

	[Install]
	WantedBy=multi-user.target

Перезапускаем *systemd*, запускаем *prometheus* и проверяем его статус

	# systemctl daemon-reload
	# systemctl start node_exporter
	# systemctl status node_exporter

	● node_exporter.service - Prometheus Node Exporter
	   Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: disabled)
	   Active: active (running) since Tue 2020-10-27 13:52:55 UTC; 1h 3min ago
	 Main PID: 4076 (node_exporter)
	   CGroup: /system.slice/node_exporter.service
	           └─4076 /usr/local/bin/node_exporter

Заходим на *http://localhost:9100* и проверяем как работает *node exporter*


### Alertmanager

Также аналогично устанавливается *alertmanager*. [Официальный сайт с релизами](https://github.com/prometheus/alertmanager/releases)

Скачиваем, распаковываем, создаем каталоги, копируем файлы, создаем пользователя, даем права на каталоги

	$ wget https://github.com/prometheus/alertmanager/releases/download/v0.21.0/alertmanager-0.21.0.linux-amd64.tar.gz
	$ tar xf alertmanager-0.21.0.linux-amd64.tar.gz
	# cd alertmanager-0.21.0.linux-amd64
	# cp alertmanager /usr/local/bin
	# mkdir /etc/alertmanager /var/lib/alertmanager
	# cp alertmanager.yml /etc/alertmanager
	# useradd --no-create-home --home-dir / --shell /bin/false alertmanager
	# chown -R alertmanager:alertmanager /var/lib/alertmanager

Создаём systemd-юнит *alertmanager*

	[Unit]
	Description=Alertmanager for prometheus
	After=network.target

	[Service]
	User=alertmanager
	ExecStart=/usr/local/bin/alertmanager \
	  --config.file=/etc/alertmanager/alertmanager.yml \
	  --storage.path=/var/lib/alertmanager/
	ExecReload=/bin/kill -HUP $MAINPID

	NoNewPrivileges=true
	ProtectHome=true
	ProtectSystem=full

	[Install]
	WantedBy=multi-user.target

Запускаем *alertmanager*

	# systemctl daemon-reload
	# systemctl start alertmanager
	# systemctl status alertmanager

	● alertmanager.service - Alertmanager for prometheus
	   Loaded: loaded (/etc/systemd/system/alertmanager.service; disabled; vendor preset: disabled)
	   Active: active (running) since Tue 2020-10-27 14:02:20 UTC; 55min ago
	 Main PID: 4195 (alertmanager)
	   CGroup: /system.slice/alertmanager.service
	           └─4195 /usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --storage.path=/var/lib/alertmanager/

Заходим на *http://localhost:9093* и проверяем как работает *alermanager*

### Grafana

В отличие от предыдущих сервисов, *grafana* устанавливается из пакета. [Официальный сайт с пакетами](https://grafana.com/grafana/download)

	$ wget https://dl.grafana.com/oss/release/grafana-7.2.2-1.x86_64.rpm
	$ yum install grafana-7.2.2-1.x86_64.rpm 

Запускаем grafana:

	# systemctl start grafana
	# systemctl status grafana

	● grafana-server.service - Grafana instance
	   Loaded: loaded (/usr/lib/systemd/system/grafana-server.service; disabled; vendor preset: disabled)
	   Active: active (running) since Tue 2020-10-27 14:07:08 UTC; 53min ago
	     Docs: http://docs.grafana.org
	 Main PID: 4499 (grafana-server)
	   CGroup: /system.slice/grafana-server.service
	           └─4499 /usr/sbin/grafana-server --config=/etc/grafana/grafana.ini --pidfile=/var/run/grafana/grafana-server.pid --packaging=rpm cfg:default.paths.logs=/var/log/grafana cfg:default.paths.data=/var/lib/grafana cfg:default.paths.plugins=/var/lib/grafana/plugins cfg:default.paths.provisioning=/etc/grafana/provisioning

### Первичная настройка

Правим `/etc/prometheus/prometheus.yml`, добавляем секцию *job_name: node*

	# my global config
	global:
	  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
	  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
	  # scrape_timeout is set to the global default (10s).

	# Alertmanager configuration
	alerting:
	  alertmanagers:
	  - static_configs:
	    - targets:
	       - localhost:9093

	# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
	rule_files:
	  # - "first_rules.yml"
	  # - "second_rules.yml"

	# A scrape configuration containing exactly one endpoint to scrape:
	# Here it's Prometheus itself.
	scrape_configs:
	  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
	  - job_name: 'prometheus'

	    # metrics_path defaults to '/metrics'
	    # scheme defaults to 'http'.

	    static_configs:
	    - targets:
	       - localhost:9090


	  - job_name: 'node'
	    static_configs:
	    - targets:
	       - localhost:9100

Перечитываем конфигурацию

	# systemctl reload prometheus

Открываем интерфейс Grafana по адресу http://localhost:3000. Логинимся (admin:admin), при желании меняем пароль. Практически всё в Графане настраивается тут же, через веб-интерфейс. Нажимаем на боковой панели Configuration (шестерёнка) → Preferences. Идём на вкладку Data Sources, добавляем наш Prometheus. Теперь добавим готовый дашборд Node exporter full. Это лучший дашборд для старта из всех, что я пересмотрел. Идём Dashboards (квадрат) → Manage → Import. По *id 1860* находим готовый дашбоард для *node exporter* и нажимаем Load. 

Вы прекрасны!!! 



P.S. Материал основан на этой [статье](https://laurvas.ru/prometheus-p1/)