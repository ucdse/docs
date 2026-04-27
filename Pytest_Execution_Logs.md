================================================ test session starts =================================================
platform win32 -- Python 3.12.12, pytest-9.0.3, pluggy-1.6.0
rootdir: D:\05_UCD\Software Engineering\bike_app\flask-app
plugins: anyio-4.12.1, langsmith-0.7.16, cov-7.1.0
collected 355 items                                                                                                   

tests\test_chat_routes.py ........................                                                              [  6%]
tests\test_chat_service.py ..................                                                                   [ 11%]
tests\test_chat_service_llm.py ........                                                                         [ 14%]
tests\test_contracts.py .............................................                                           [ 26%]
tests\test_email_utils.py ............                                                                          [ 30%]
tests\test_journey_routes.py ..................                                                                 [ 35%]
tests\test_journey_service.py ........                                                                          [ 37%]
tests\test_journey_service_matrix.py ......                                                                     [ 39%]
tests\test_prediction_service.py .........                                                                      [ 41%]
tests\test_schemas.py .................................................                                         [ 55%]
tests\test_station_routes.py ............                                                                       [ 58%]
tests\test_station_service.py ..............                                                                    [ 62%]
tests\test_user_routes.py .................................                                                     [ 72%]
tests\test_user_service.py .............................................................                        [ 89%]
tests\test_utils.py ................                                                                            [ 93%] 
tests\test_weather_routes.py .....                                                                              [ 95%]
tests\test_weather_routes_validation.py ......                                                                  [ 96%]
tests\test_weather_service.py ...........                                                                       [100%]

================================================== warnings summary ================================================== 
tests/test_chat_routes.py: 15 warnings
tests/test_user_routes.py: 5 warnings
tests/test_user_service.py: 10 warnings
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\jwt\api_jwt.py:153: InsecureKeyLengthWarning: The HMAC key is 28 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.      
    return self._jws.encode(

tests/test_chat_routes.py: 15 warnings
tests/test_user_routes.py: 3 warnings
tests/test_user_service.py: 7 warnings
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\jwt\api_jwt.py:371: InsecureKeyLengthWarning: The HMAC key is 28 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.      
    decoded = self.decode_complete(

tests/test_prediction_service.py::TestGetStationPredictions::test_raises_prediction_error_when_no_forecasts
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_returned_for_valid_station_and_forecasts 
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_clamped_to_zero_minimum
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_clamped_to_bike_stands_maximum
  D:\05_UCD\Software Engineering\bike_app\flask-app\app\services\prediction_service.py:60: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    now = datetime.utcnow()

tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_returned_for_valid_station_and_forecasts 
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_returned_for_valid_station_and_forecasts 
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_returned_for_valid_station_and_forecasts 
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_clamped_to_zero_minimum
tests/test_prediction_service.py::TestGetStationPredictions::test_predictions_clamped_to_bike_stands_maximum
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_prediction_service.py:36: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    f.forecast_time = datetime.utcnow().replace(

tests/test_user_service.py::TestCreateToken::test_creates_valid_jwt
tests/test_user_service.py::TestCreateToken::test_token_contains_version
tests/test_user_service.py::TestDecodeToken::test_valid_token_decoded_successfully
tests/test_user_service.py::TestDecodeToken::test_expired_token_raises_auth_error
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\jwt\api_jwt.py:153: InsecureKeyLengthWarning: The HMAC key is 6 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.       
    return self._jws.encode(

tests/test_user_service.py::TestCreateToken::test_creates_valid_jwt
tests/test_user_service.py::TestCreateToken::test_token_contains_version
tests/test_user_service.py::TestDecodeToken::test_valid_token_decoded_successfully
tests/test_user_service.py::TestDecodeToken::test_expired_token_raises_auth_error
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\jwt\api_jwt.py:371: InsecureKeyLengthWarning: The HMAC key is 6 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.       
    decoded = self.decode_complete(

tests/test_user_service.py::TestDecodeToken::test_wrong_secret_raises_auth_error
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\jwt\api_jwt.py:153: InsecureKeyLengthWarning: The HMAC key is 14 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.      
    return self._jws.encode(

tests/test_user_service.py::TestDecodeToken::test_wrong_secret_raises_auth_error
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\jwt\api_jwt.py:371: InsecureKeyLengthWarning: The HMAC key is 12 bytes long, which is below the minimum recommended length of 32 bytes for SHA256. See RFC 7518 Section 3.2.      
    decoded = self.decode_complete(

tests/test_weather_routes.py: 1 warning
tests/test_weather_service.py: 9 warnings
  D:\05_UCD\Software Engineering\bike_app\flask-app\app\services\weather_service.py:25: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    now = datetime.utcnow()

tests/test_weather_service.py::TestGetWeather::test_returns_current_and_hourly_keys
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:25: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1)

tests/test_weather_service.py: 12 warnings
  C:\Users\USER\miniconda3\envs\flask-app\Lib\site-packages\sqlalchemy\sql\schema.py:3624: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    return util.wrap_callable(lambda ctx: fn(), fn)  # type: ignore

tests/test_weather_service.py::TestGetWeather::test_current_contains_expected_fields
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:34: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_list_contains_all_forecasts
tests/test_weather_service.py::TestGetWeather::test_hourly_list_contains_all_forecasts
tests/test_weather_service.py::TestGetWeather::test_hourly_list_contains_all_forecasts
tests/test_weather_service.py::TestGetWeather::test_hourly_list_contains_all_forecasts
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:51: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=i)

tests/test_weather_service.py::TestGetWeather::test_current_uses_earliest_forecast
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:60: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_current_uses_earliest_forecast
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:64: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=2),

tests/test_weather_service.py::TestGetWeather::test_temperature_value_preserved
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:73: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_past_forecasts_excluded
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:83: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() - timedelta(hours=2)

tests/test_weather_service.py::TestGetWeather::test_weather_code_in_current
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:91: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1),
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:105: DeprecationWarning: datetime.dat    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:105: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
    forecast_time=datetime.utcnow() + timedelta(hours=1),

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field

tests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:105: DeprecationWarning: datetime.dattests/test_weather_service.py::TestGetWeather::test_hourly_items_contain_pop_field
  D:\05_UCD\Software Engineering\bike_app\flask-app\tests\test_weather_service.py:105: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
    forecast_time=datetime.utcnow() + timedelta(hours=1),

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html
========================================= 355 passed, 108 warnings in 13.04s ========================================= 