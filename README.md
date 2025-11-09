# WeatherNotifier


public class Main {

public static void main(String[] args) {
    // Constants :-

    String MY_NUM = "Your Num";
    double MY_LAT = 34.083672;
    double MY_LONG = 74.797279;

    String API_KEY = "Your key";
    String ACCOUNT_SID = "Your SID";
    String AUTH_TOKEN = "Your Token";
    try {
        // Fetching  Weather Data :-

        String urlWeather = String.format(
                "https://api.openweathermap.org/data/2.5/forecast?lat=%f&lon=%f&appid=%s&cnt=4",
                MY_LAT, MY_LONG, API_KEY);
        // Creating a Url and an open connection :-
        URL url = new URL(urlWeather);

        // Java lib to send a get request....
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        // Since UrlConnection is a method which belongs to url class and returns
        // a general URLConnection object but The openConnection() method can return different subclasses
        // depending on the protocol. Hence, we need to typecast it to httpurlconnection......
        conn.setRequestMethod("GET");

        int responseCode = conn.getResponseCode();
        if (responseCode != 200) {
            throw new RuntimeException("Oops Your login failed  : " + responseCode);
        }

        // BufferedReader - method to read HTTP response one line at a time..
        BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        // conn.getInputStream get the data from the server into JSON Format..
        StringBuilder response = new StringBuilder();
        // Collects all that data lines into a single big string.. since strings are immutable objects
        // in java...
        String line;
        while ((line = br.readLine()) != null) {
            response.append(line);
        }
        br.close();

        conn.disconnect();

       // System.out.println(response);


        // Parsing the Weather JSON Data....

        JSONObject weatherData = new JSONObject(response.toString());
        JSONArray list = weatherData.getJSONArray("list");

        boolean willRain = false;
        for (int i = 0; i < list.length(); i++) {
            JSONObject hourData = list.getJSONObject(i);
            JSONArray weatherArray = hourData.getJSONArray("weather");
            int conditionCode = weatherArray.getJSONObject(0).getInt("id");

            if (conditionCode < 700) {
                willRain = true;
                break;
            }
        }

        // Sending SMS via Twilio....
        Twilio.init(ACCOUNT_SID, AUTH_TOKEN);

        if (willRain) {
            Message message = Message.creator(

                    new com.twilio.type.PhoneNumber("+91" + MY_NUM),
                    new com.twilio.type.PhoneNumber("+19788438530"),
                    "It's going to rain today. Remember to bring an Umbrella☔️"
            ).create();
            System.out.println(message.getBody());
        } else {
            Message message = Message.creator(
                    new com.twilio.type.PhoneNumber("+91" + MY_NUM),
                    new com.twilio.type.PhoneNumber("+19788438530"),
                    "It's gonna be a Sunny Day. Enjoy the Apricity🌅"
            ).create();
            System.out.println(message.getBody());
        }

    } catch (Exception e) {
        e.printStackTrace();
    }
}
