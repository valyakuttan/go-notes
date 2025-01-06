# Singleflight And Multiple Requests

When multiple requests at the same time asking for the same data,
**singleflight** ensures that only one of them is doing the actual work and
the rest share the result.

```go

// ...

/ A struct to hold the singleflight group and a cache.
type WeatherService struct {
    requestGroup singleflight.Group
    cache        sync.Map
}

// Function to simulate fetching weather data from an external service.
func (w *WeatherService) fetchWeatherData(city string) (string, error) {
    time.Sleep(200 * time.Millisecond)
    return fmt.Sprintf("weather at %s", city), nil
}

// Function to get weather data with caching and singleflight control.
func (w *WeatherService) GetWeather(city string) (string, error) {
    // First, check if the data is already in the cache.
    if data, ok := w.cache.Load(city); ok {
        return data.(string), nil
    }

    // If not, use singleflight to ensure only one fetch is happening for the same city.
    data, err, _ := w.requestGroup.Do(city, func() (any, error) {
        // Fetch the data.
        result, err := w.fetchWeatherData(city)
        if err == nil {
            // Store the result in the cache.
            w.cache.Store(city, result)
        }
        return result, err
    })

    if err != nil {
        return "", err
    }
    return data.(string), nil
}

func main() {
    var wg sync.WaitGroup
    service := &WeatherService{}

    // Simulate multiple concurrent requests for the same city.
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            weather, err := service.GetWeather("NewYork")
            if err == nil {
                fmt.Printf("Goroutine %d got weather data: %s\n", i, weather)
            } else {
                fmt.Printf("Goroutine %d encountered an error: %s\n", i, err)
            }
        }(i)
    }

    wg.Wait()
}

// ...

```
