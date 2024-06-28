# Weather and air-quality operations alert

This workflow checks a location's next-day weather and current air quality, evaluates explicit operational thresholds, and optionally posts an alert to Slack. The production data path uses Open-Meteo and does not require an API key for its open-access, non-commercial evaluation/prototyping tier.

## Architecture

`Webhook or disabled schedule → normalize coordinates/thresholds → test fixture or Open-Meteo APIs → deterministic risk engine → alert text → mock/Slack/skip → receipt`

Signals currently supported:

- maximum temperature in °C;
- maximum precipitation probability;
- maximum wind speed in km/h;
- U.S. air-quality index.

The risk engine returns every triggered rule, a combined severity, the evaluated values, and the thresholds used. This makes an alert explainable and easy to audit.

## Run the included test

```powershell
$body = Get-Content -Raw .\6-weather-operations-alert\fixtures\severe-manila-weather.json
Invoke-RestMethod -Method Post -Uri http://localhost:5678/webhook/weather-operations -ContentType application/json -Body $body
```

Expected: `severity=high`, `alert_count=3`, and `notification.status=mocked`. Heat, rain, and AQI trigger; the wind value remains below its configured limit.

## Production configuration

- Set `WEATHER_LOCATION`, `WEATHER_LATITUDE`, and `WEATHER_LONGITUDE`.
- Tune thresholds in the webhook body or in `Normalize Weather Request`.
- Configure `SLACK_WEBHOOK_URL` if notifications are wanted.
- Enable `Morning Weather Schedule` only after testing the real location.

Open-Meteo's official pricing page currently describes the open-access API as non-commercial, rate-limited, and without an uptime guarantee. It also requires attribution for the underlying CC BY 4.0 data. Review [Open-Meteo pricing and licence terms](https://open-meteo.com/en/pricing) before commercial use.

For safety-critical operations, do not use this automation as the only warning source. Add an approved meteorological source, monitoring of failed executions, stale-data detection, and an escalation path outside Slack.
