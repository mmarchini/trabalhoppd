As principais diferenças entre a implementação abaixo e o exemplo apresentado em aula utilizando **barreiras**, são:

* Quando uma thread termina o processamento da linha, ela notifica o Solver que terminou os seus cálculos, e depois disso finaliza a sua execução
  * No exemplo apresentado em aula, as threads precisam esperar todas as outras finalizarem, para só então finalizar a sua própria execução
* o Solver espera as threads finalizarem as suas execuções, e então ele mesmo executa o *merge* das linhas
  * No exemplo, a thread disparada pela barreira executa o *merge*, e o Solver espera a execução das thread através do método `waitUntilDone`

Utilizei comentários no código para explicar melhor a minha solução. Os comentários originais do exemplo foram removidos para facilitar a visualziação dos trechos de código modificados.


```java
class Worker implements Runnable {
    int myRow;
    int mySolver;

    public Worker(int row, Solver solver) { myRow = row; mySolver = solver}

    public void run() {
        processRow(myRow);
        // Notifica o Solver que essa thread terminou a execução
        solver.workerFinished();
    }
}

class Solver {
    // Variável que vai monitorar para ver quantas threads faltam finalizar para terminar os cálculos
    private int remainingThreads;
    final int N;
    final float[][] data;

    public Solver(float[][] matrix) {
        data = matrix;
        N = matrix.length;

        // Inicializa a variável que monitora quantas threads faltam finalizar para terminar os cálculos    
        synchronized (this) {
            remainingThreads = N;
        }
        
        for (int i = 0; i < N; ++i)
            new Thread(new Worker(i, this)).start();

        // Espera a variável que monitora chegar a zero
        while(remainingThreads != 0);
        
        // Quando a variável chegar a zero, as threads terminaram de executar
        // Damos merge nas linhas
        mergeRows();
    }

    // Esse método é responsável por decrementar o valor da variável que monitora as theads
    // Ele é executado pelas threads, após elas terminarem de realizar os cálculos e antes delas finalizarem
    public synchronized void workerFinished(){
        remainingThreads--;
    }
}
```

