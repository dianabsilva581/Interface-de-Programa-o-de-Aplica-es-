# Interface-de-Programa-o-de-Aplica-es-
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
         
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>currency-converter</artifactId>
    <version>1.0-SNAPSHOT</version>
    
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
    
    <dependencies>
        <!-- Dependência para fazer requisições HTTP -->
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>4.5.13</version>
        </dependency>
        
        <!-- Dependência para parsear JSON -->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.13.1</version>
        </dependency>
    </dependencies>
</project>
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class CurrencyConverter {

    private static final String API_URL = "https://api.exchangerate-api.com/v4/latest/USD"; // Exemplo de API

    private Map<String, Double> exchangeRates;

    public CurrencyConverter() {
        this.exchangeRates = new HashMap<>();
    }

    public void fetchExchangeRates() throws IOException {
        CloseableHttpClient httpClient = HttpClients.createDefault();
        HttpGet request = new HttpGet(API_URL);
        CloseableHttpResponse response = httpClient.execute(request);

        try {
            HttpEntity entity = response.getEntity();
            if (entity != null) {
                InputStream inputStream = entity.getContent();
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
                String line;
                StringBuilder result = new StringBuilder();
                while ((line = reader.readLine()) != null) {
                    result.append(line);
                }
                
                // Parse JSON response
                ObjectMapper mapper = new ObjectMapper();
                JsonNode jsonNode = mapper.readTree(result.toString());
                
                // Extract exchange rates from JSON
                JsonNode ratesNode = jsonNode.path("rates");
                for (String currency : ratesNode.fieldNames()) {
                    double rate = ratesNode.get(currency).asDouble();
                    exchangeRates.put(currency, rate);
                }
            }
        } finally {
            response.close();
        }
    }

    public double convertCurrency(double amount, String fromCurrency, String toCurrency) {
        double fromRate = exchangeRates.getOrDefault(fromCurrency, 1.0);
        double toRate = exchangeRates.getOrDefault(toCurrency, 1.0);
        return (amount / fromRate) * toRate;
    }

    public static void main(String[] args) {
        CurrencyConverter converter = new CurrencyConverter();
        Scanner scanner = new Scanner(System.in);

        try {
            converter.fetchExchangeRates();
            System.out.println("Taxas de câmbio carregadas com sucesso.");

            while (true) {
                System.out.println("\nEscolha uma opção:");
                System.out.println("1. USD para EUR");
                System.out.println("2. USD para GBP");
                System.out.println("3. EUR para USD");
                System.out.println("4. EUR para GBP");
                System.out.println("5. GBP para USD");
                System.out.println("6. GBP para EUR");
                System.out.println("0. Sair");

                int choice = scanner.nextInt();
                if (choice == 0) {
                    System.out.println("Saindo...");
                    break;
                }

                double amount;
                switch (choice) {
                    case 1:
                        System.out.print("Digite o valor em USD: ");
                        amount = scanner.nextDouble();
                        System.out.println("Valor em EUR: " + converter.convertCurrency(amount, "USD", "EUR"));
                        break;
                    case 2:
                        System.out.print("Digite o valor em USD: ");
                        amount = scanner.nextDouble();
                        System.out.println("Valor em GBP: " + converter.convertCurrency(amount, "USD", "GBP"));
                        break;
                    case 3:
                        System.out.print("Digite o valor em EUR: ");
                        amount = scanner.nextDouble();
                        System.out.println("Valor em USD: " + converter.convertCurrency(amount, "EUR", "USD"));
                        break;
                    case 4:
                        System.out.print("Digite o valor em EUR: ");
                        amount = scanner.nextDouble();
                        System.out.println("Valor em GBP: " + converter.convertCurrency(amount, "EUR", "GBP"));
                        break;
                    case 5:
                        System.out.print("Digite o valor em GBP: ");
                        amount = scanner.nextDouble();
                        System.out.println("Valor em USD: " + converter.convertCurrency(amount, "GBP", "USD"));
                        break;
                    case 6:
                        System.out.print("Digite o valor em GBP: ");
                        amount = scanner.nextDouble();
                        System.out.println("Valor em EUR: " + converter.convertCurrency(amount, "GBP", "EUR"));
                        break;
                    default:
                        System.out.println("Opção inválida. Escolha novamente.");
                        break;
                }
            }
        } catch (IOException e) {
            System.err.println("Erro ao obter taxas de câmbio: " + e.getMessage());
        } finally {
            scanner.close();
        }
    }
}
