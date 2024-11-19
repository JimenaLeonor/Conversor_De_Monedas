import java.io.IOException;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import com.google.gson.Gson;
import com.google.gson.JsonObject;
import java.util.Scanner;

public class ConversorDeMonedas {
    private static final Gson gson = new Gson();
    private static final String API_URL = "https://api.exchangerate-api.com/v4/latest/USD";

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        boolean continuar = true;

        System.out.println("*****************************************************");
        System.out.println("Bienvenido/a al Conversor de Moneda =]");
        while (continuar) {
            mostrarMenu();
            int opcion = scanner.nextInt();

            if (opcion >= 1 && opcion <= 6) {
                System.out.print("Ingrese el valor que desea convertir: ");
                double cantidad = scanner.nextDouble();

                switch (opcion) {
                    case 1 -> convertirMoneda("USD", "ARS", cantidad); // Dólar a Peso Argentino
                    case 2 -> convertirMoneda("ARS", "USD", cantidad); // Peso Argentino a Dólar
                    case 3 -> convertirMoneda("USD", "BRL", cantidad); // Dólar a Real Brasileño
                    case 4 -> convertirMoneda("BRL", "USD", cantidad); // Real Brasileño a Dólar
                    case 5 -> convertirMoneda("USD", "COP", cantidad); // Dólar a Peso Colombiano
                    case 6 -> convertirMoneda("COP", "USD", cantidad); // Peso Colombiano a Dólar
                }
            } else if (opcion == 7) {
                System.out.println("Saliendo del conversor de moneda. ¡Hasta luego!");
                continuar = false;
            } else {
                System.out.println("Por favor, elija una opción válida.");
            }
        }
        scanner.close();
    }

    private static void mostrarMenu() {
        System.out.println("\nOpciones de Conversión:");
        System.out.println("1) Dólar =>> Peso argentino");
        System.out.println("2) Peso argentino =>> Dólar");
        System.out.println("3) Dólar =>> Real brasileño");
        System.out.println("4) Real brasileño =>> Dólar");
        System.out.println("5) Dólar =>> Peso colombiano");
        System.out.println("6) Peso colombiano =>> Dólar");
        System.out.println("7) Salir");
        System.out.print("Elija una opción válida: ");
    }

    private static void convertirMoneda(String monedaOrigen, String monedaDestino, double cantidad) {
        try {
            JsonObject rates = obtenerTasasDeCambio(monedaOrigen);
            if (rates != null && rates.has(monedaDestino)) {
                double tasaDestino = rates.get(monedaDestino).getAsDouble();
                double conversion = cantidad * tasaDestino;
                System.out.printf("El valor %.2f [%s] corresponde al valor final de %.2f [%s]%n", cantidad, monedaOrigen, conversion, monedaDestino);
            } else {
                System.out.println("No se encontró la tasa de cambio para la moneda de destino.");
            }
        } catch (IOException | InterruptedException e) {
            System.out.println("Error al obtener las tasas de cambio: " + e.getMessage());
        }
    }

    private static JsonObject obtenerTasasDeCambio(String monedaBase) throws IOException, InterruptedException {
        HttpClient client = HttpClient.newHttpClient();
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create(API_URL))
                .build();

        HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());

        if (response.statusCode() == 200) {
            JsonObject jsonResponse = gson.fromJson(response.body(), JsonObject.class);
            return jsonResponse.getAsJsonObject("rates");
        } else {
            System.out.println("Error: No se pudo obtener la respuesta de la API.");
            return null;
        }
    }
}
