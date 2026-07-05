# i2i-Academy-ApacheIngite-1

This project is about Apache Ignite tecnology and this technology's application on telecommunication systems. 
Main goal is to determine customer id and random values for data, sms and call usage.

Firstly we have to run our Apache Ignite 3 database with docker. After creating the container and running the Apache Ignite we are going to connect this container with our Java project.

XML File:
We have to pull Apache database to connect with our project. Because of this we added dependency in our XML file.

    <dependencies>
        <dependency>
            <groupId>org.apache.ignite</groupId>
            <artifactId>ignite-client</artifactId>
            <version>3.1.0</version>
        </dependency>
    </dependencies>

Main File:
To connect my java project and ignite container I built main file. The connection successfully
happened from 10800 port. Then I organized structure to create a table to hold subscriber’s usage
information and clean all data for every simulation.

    import org.apache.ignite.client.IgniteClient;
    import org.apache.ignite.sql.IgniteSql;
    
    public class Main {
        public static void main(String[] args) {
            System.out.println("Connecting to Ignite server...");
    
    
            try (IgniteClient client = IgniteClient.builder().addresses("127.0.0.1:10800").build()) {
    
                System.out.println("Connection Successfull!");
    
    
                IgniteSql sql = client.sql();
    
    
                sql.execute(null,
                        "CREATE TABLE IF NOT EXISTS Subscriber (" +
                                "customerId INT PRIMARY KEY, " +
                                "dataUsage DOUBLE, " +
                                "smsUsage INT, " +
                                "callUsage INT)"
                );
                System.out.println("Subscriber table is ready.");
    
    
                sql.execute(null, "DELETE FROM Subscriber");
                System.out.println("Old data cleared, table is ready to new simulation.");
    
                System.out.println("First step finished successfully!");
    
            } catch (Exception e) {
                System.err.println("An error occured: " + e.getMessage());
            }
        }
    }



Subscriber File:
I determined the data about every subscriber’s usage.

    public class Subscriber {
    
        Integer customerId;
    
    
        Double dataUsage;
        //MB or GB
    
        Integer smsUsage;
    
        Integer callUsage;
    }

IgniteSimulation File:
In this file I created a simple simulation to track subscriber data usage. First the code connects to the
local Apache Ignite node and creates a table named Subscriber. It deletes any old data so we start
fresh every time we run the program. Then I wrote a loop to insert five dummy subscribers with zero
starting values. After that the code generates random data sms and call usage amounts and updates
the database using standard SQL queries. At the end it just fetches the final table and prints all the
updated subscriber stats to the console to show everything works perfectly.

    import org.apache.ignite.client.IgniteClient;
    import org.apache.ignite.sql.IgniteSql;
    import org.apache.ignite.sql.ResultSet;
    import org.apache.ignite.sql.SqlRow;
    import java.util.Random; // Rastgele sayılar üretmek için kullanacağımız kütüphane
    
    public class IgniteSimulation {
        public static void main(String[] args) {
            System.out.println("Connecting to Ignite server...");
    
            try (IgniteClient client = IgniteClient.builder().addresses("127.0.0.1:10800").build()) {
    
                System.out.println("Connection successful!");
                IgniteSql sql = client.sql();
    
                sql.execute(null,
                        "CREATE TABLE IF NOT EXISTS Subscriber (" +
                                "customerId INT PRIMARY KEY, " +
                                "dataUsage DOUBLE, " +
                                "smsUsage INT, " +
                                "callUsage INT)"
                );
                sql.execute(null, "DELETE FROM Subscriber");
                System.out.println("Old data cleared, ready for the new simulation.");
    
    
                System.out.println("Inserting 5 dummy subscribers with zero usage...");
                for (int i = 1; i <= 5; i++) {
    
                    sql.execute(null,
                            "INSERT INTO Subscriber (customerId, dataUsage, smsUsage, callUsage) VALUES (?, ?, ?, ?)",
                            i, 0.0, 0, 0
                    );
                }
                System.out.println("Subscribers inserted successfully.");
    
    
                System.out.println("Simulating random usage updates...");
                Random random = new Random();
    
                for (int i = 1; i <= 5; i++) {
    
                    double addedData = Math.round(random.nextDouble() * 10.0 * 100.0) / 100.0; // 0 ile 10 GB arası
                    int addedSms = random.nextInt(50);   // 0 ile 50 SMS arası
                    int addedCalls = random.nextInt(100); // 0 ile 100 Dakika arası
    
    
                    sql.execute(null,
                            "UPDATE Subscriber SET dataUsage = dataUsage + ?, smsUsage = smsUsage + ?, callUsage = callUsage + ? WHERE customerId = ?",
                            addedData, addedSms, addedCalls, i
                    );
                }
                System.out.println("Usage updates completed.");
    
    
                System.out.println("\n--- FINAL SUBSCRIBER STATES ---");
    
    
                ResultSet<SqlRow> results = sql.execute(null, "SELECT * FROM Subscriber");
    
    
                while (results.hasNext()) {
                    SqlRow row = results.next();
                    System.out.println("Customer ID: " + row.intValue("customerId") +
                            " | Data: " + row.doubleValue("dataUsage") + " GB" +
                            " | SMS: " + row.intValue("smsUsage") +
                            " | Calls: " + row.intValue("callUsage") + " mins");
                }
    
                System.out.println("\nSimulation finished successfully!");
    
            } catch (Exception e) {
                System.err.println("An error occurred: " + e.getMessage());
            }
        }
    }



At the end I successfully run the Ignite and got my table results perfectly.
