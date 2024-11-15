package comunicacionconfiable;

import java.io.*;
import java.net.*;
import java.util.ArrayList;
import java.util.List;

public class ServidorMulticast {
    
    private static final int TCP_PORT = 6000;
    private static final int UDP_PORT = 5000;
    private static List<InetAddress> clientes = new ArrayList<>();

    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(TCP_PORT)) {
            System.out.println("Servidor TCP/UDP iniciado.");

            // Hilo para aceptar conexiones TCP y registrar clientes
            new Thread(() -> {
                try {
                    while (true) {
                        Socket clientSocket = serverSocket.accept();
                        InetAddress clientAddress = clientSocket.getInetAddress();
                        System.out.println("Cliente conectado desde " + clientAddress);
                        clientes.add(clientAddress);

                        BufferedReader in = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
                        String mensaje = in.readLine();
                        System.out.println("Mensaje recibido (TCP): " + mensaje);

                        // Enviar confirmación al cliente vía TCP
                        PrintWriter out = new PrintWriter(clientSocket.getOutputStream(), true);
                        out.println("Mensaje recibido correctamente por TCP.");
                        clientSocket.close();
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }).start();

            // Enviar mensajes a todos los clientes registrados usando UDP
            DatagramSocket udpSocket = new DatagramSocket();
            String udpMensaje = "Mensaje desde el servidor vía UDP";
            byte[] buffer = udpMensaje.getBytes();

            while (true) {
                for (InetAddress clientAddress : clientes) {
                    DatagramPacket packet = new DatagramPacket(buffer, buffer.length, clientAddress, UDP_PORT);
                    udpSocket.send(packet);
                    System.out.println("Mensaje enviado a cliente (UDP) en IP: " + clientAddress.getHostAddress());
                }
                Thread.sleep(5000); // Reenvía el mensaje cada 5 segundos
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
