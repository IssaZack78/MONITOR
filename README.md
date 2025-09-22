# Monitor

java

public class Main {

    // Monitor
    static class Monitor {
        private int value = 0;

        // Incrementar: metodo sincronizado
        public synchronized void incrementar() {
            value++;
            // notificar en caso de que haya hilos esperando para decrementar
            notifyAll();
        }

        // Decrementar: espera si value == 0 para no bajar a negativo
        public synchronized void decrementar() throws InterruptedException {
            while (value == 0) {
                wait();
            }
            value--;
        }

        public synchronized int getValue() {
            return value;
        }
    }

    // Hilo que incrementa
    static class Incrementer implements Runnable {
        private final Monitor monitor;
        private final int veces;
        private final int delayMs;

        public Incrementer(Monitor monitor, int veces, int delayMs) {
            this.monitor = monitor;
            this.veces = veces;
            this.delayMs = delayMs;
        }

        @Override
        public void run() {
            for (int i = 0; i < veces; i++) {
                monitor.incrementar();
                System.out.println("Incrementador -> " + monitor.getValue());
                try {
                    Thread.sleep(delayMs);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
    }

    // Hilo que decrementa
    static class Decrementer implements Runnable {
        private final Monitor monitor;
        private final int veces;
        private final int delayMs;

        public Decrementer(Monitor monitor, int veces, int delayMs) {
            this.monitor = monitor;
            this.veces = veces;
            this.delayMs = delayMs;
        }

        @Override
        public void run() {
            for (int i = 0; i < veces; i++) {
                try {
                    monitor.decrementar();
                    System.out.println("Decrementador -> " + monitor.getValue());
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }

                try {
                    Thread.sleep(delayMs);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
    }

    // Main: crea el monitor y los hilos
    public static void main(String[] args) throws InterruptedException {
        Monitor monitor = new Monitor();

        // Ajusta 'veces' y delays segun quieras
        int veces = 50;
        Thread inc = new Thread(new Incrementer(monitor, veces, 10));
        Thread dec = new Thread(new Decrementer(monitor, veces, 15));

        inc.start();
        dec.start();

        inc.join();
        dec.join();

        System.out.println("Final value: " + monitor.getValue());
    }
}
