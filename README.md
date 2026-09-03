# otel-docker

## Alertas por Slack con Alertmanager

El stack actual no incluye Alertmanager ni reglas de alertas. Para recibir un aviso en Slack cuando una condición se mantenga durante un tiempo determinado, hay que configurar estos componentes:

1. En Slack, crea una aplicación con un **Incoming Webhook**, selecciona el canal de destino y guarda la URL del webhook como un secreto. No la publiques en el repositorio.
2. Añade un archivo `observability/alertmanager.yml` con el receptor de Slack:

	 ```yaml
	 route:
		 receiver: slack
		 group_wait: 30s
		 group_interval: 5m
		 repeat_interval: 4h

	 receivers:
		 - name: slack
			 slack_configs:
				 - api_url: ${SLACK_WEBHOOK_URL}
					 channel: '#alertas'
					 send_resolved: true
	 ```

3. Añade las reglas de Prometheus en `observability/alert-rules.yml`. El campo `for` determina cuánto debe mantenerse la condición antes de que Prometheus envíe la alerta a Alertmanager:

	 ```yaml
	 groups:
		 - name: infraestructura
			 rules:
				 - alert: CollectorDown
					 expr: up{job="otel-collector"} == 0
					 for: 2m
					 labels:
						 severity: critical
					 annotations:
						 summary: "OpenTelemetry Collector no disponible"
						 description: "El Collector lleva caído más de 2 minutos."
	 ```

4. En `observability/prometheus.yml`, registra Alertmanager y las reglas:

	 ```yaml
	 alerting:
		 alertmanagers:
			 - static_configs:
					 - targets: ['alertmanager:9093']

	 rule_files:
		 - /etc/prometheus/alert-rules.yml
	 ```

5. Añade Alertmanager al `docker-compose.yml` y monta su configuración:

	 ```yaml
	 alertmanager:
		 image: prom/alertmanager:v0.28.1
		 environment:
			 SLACK_WEBHOOK_URL: ${SLACK_WEBHOOK_URL}
		 volumes:
			 - ./observability/alertmanager.yml:/etc/alertmanager/alertmanager.yml:ro
			 - alertmanager_data:/alertmanager
		 command:
			 - --config.file=/etc/alertmanager/alertmanager.yml
			 - --config.expand-env
			 - --storage.path=/alertmanager
		 networks:
			 - observability
	 ```

	 Monta también `alert-rules.yml` en Prometheus y declara el volumen `alertmanager_data`.

6. Define `SLACK_WEBHOOK_URL` en un archivo `.env` local, excluido de Git:

	 ```dotenv
	 SLACK_WEBHOOK_URL=https://hooks.slack.com/services/REEMPLAZAR
	 ```

7. Recrea el stack y comprueba el estado:

	 ```bash
	 docker compose up -d
	 docker compose ps
	 docker compose logs -f alertmanager
	 ```

`for: 2m` evita avisos por caídas breves. `group_wait: 30s` añade una espera antes de enviar la primera notificación agrupada. Ajusta ambos valores según la urgencia de cada alerta. Para probar la integración, detén temporalmente el Collector y espera a que transcurran los 2 minutos definidos en la regla.