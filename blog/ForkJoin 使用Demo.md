### 示列1
``` java
public class CountTest {
    public static void main(String[] args) throws InterruptedException, ExecutionException {

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        //创建一个计算任务，计算 由1加到12
        CountTask countTask = new CountTask(1, 12);
        Future<Integer> future = forkJoinPool.submit(countTask);
        System.out.println("最终的计算结果：" + future.get());
    }
}

class CountTask extends RecursiveTask<Integer> {

    private static final int THRESHOLD = 2;
    private int start;
    private int end;


    public CountTask(int start, int end) {
        this.start = start;
        this.end = end;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        boolean canCompute = (end - start) <= THRESHOLD;

        //任务已经足够小，可以直接计算，并返回结果
        if (canCompute) {
            for (int i = start; i <= end; i++) {
                sum += i;
            }
            System.out.println("执行计算任务，计算    " + start + "到 " + end + "的和  ，结果是：" + sum + "   执行此任务的线程：" + Thread.currentThread().getName());

        } else { //任务过大，需要切割
            System.out.println("任务过大，切割的任务：  " + start + "加到 " + end + "的和       执行此任务的线程：" + Thread.currentThread().getName());
            int middle = (start + end) / 2;
            //切割成两个子任务
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle + 1, end);
            //执行子任务
            leftTask.fork();
            rightTask.fork();
            //等待子任务的完成，并获取执行结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            //合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }
}
```

执行结果
```
任务过大，切割的任务： 1加到 12的和 执行此任务的线程：ForkJoinPool-1-worker-1
任务过大，切割的任务： 7加到 12的和 执行此任务的线程：ForkJoinPool-1-worker-3
任务过大，切割的任务： 1加到 6的和 执行此任务的线程：ForkJoinPool-1-worker-2
执行计算任务，计算 7到 9的和 ，结果是：24 执行此任务的线程：ForkJoinPool-1-worker-3
执行计算任务，计算 1到 3的和 ，结果是：6 执行此任务的线程：ForkJoinPool-1-worker-1
执行计算任务，计算 4到 6的和 ，结果是：15 执行此任务的线程：ForkJoinPool-1-worker-1
执行计算任务，计算 10到 12的和 ，结果是：33 执行此任务的线程：ForkJoinPool-1-worker-3
最终的计算结果：78
```

示列2
```java
public class ForkJoinTask extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 5;
    private List<User> userList;

    public ForkJoinTask(List<User> userList) {
        this.userList = userList;
    }

    @Override
    protected Integer compute() {
        int sum = 0;
        if (userList == null)
            return sum;
        boolean canCompute = userList.size() <= THRESHOLD;
        //任务已经足够小，可以直接计算，并返回结果
        if (canCompute) {
            for (int i = 0; i < userList.size(); i++) {
                sum += userList.get(i).getAge();
            }
            System.out.println("执行计算任务，用户数：" + userList.size() + "，结果是：" + sum + "   执行此任务的线程：" + Thread.currentThread().getName());

        } else { //任务过大，需要切割
            System.out.println("任务过大，用户数:" + userList.size() + "，切割的任务：执行此任务的线程：" + Thread.currentThread().getName());
            int middle = userList.size() / 2;
            //切割成两个子任务
            ForkJoinTask leftTask = new ForkJoinTask(userList.subList(0, middle));
            ForkJoinTask rightTask = new ForkJoinTask(userList.subList(middle, userList.size()));
            //执行子任务
            leftTask.fork();
            rightTask.fork();
            //等待子任务的完成，并获取执行结果
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            //合并子任务
            sum = leftResult + rightResult;
        }
        return sum;
    }
}
```

执行结果：
```
任务过大，用户数:100，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-9
任务过大，用户数:50，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-9
任务过大，用户数:25，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-11
任务过大，用户数:50，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-2
任务过大，用户数:25，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-4
任务过大，用户数:12，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-13
任务过大，用户数:12，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-6
任务过大，用户数:13，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-9
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-13
任务过大，用户数:25，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-15
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-11
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-8
执行计算任务，用户数：3，结果是：24   执行此任务的线程：ForkJoinPool-1-worker-8
任务过大，用户数:25，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-1
任务过大，用户数:7，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-12
任务过大，用户数:13，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-2
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-4
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-9
执行计算任务，用户数：3，结果是：42   执行此任务的线程：ForkJoinPool-1-worker-9
执行计算任务，用户数：3，结果是：51   执行此任务的线程：ForkJoinPool-1-worker-9
任务过大，用户数:7，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-5
执行计算任务，用户数：3，结果是：174   执行此任务的线程：ForkJoinPool-1-worker-4
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-2
执行计算任务，用户数：3，结果是：192   执行此任务的线程：ForkJoinPool-1-worker-2
执行计算任务，用户数：3，结果是：60   执行此任务的线程：ForkJoinPool-1-worker-12
任务过大，用户数:12，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-1
任务过大，用户数:13，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-3
执行计算任务，用户数：3，结果是：33   执行此任务的线程：ForkJoinPool-1-worker-8
执行计算任务，用户数：3，结果是：165   执行此任务的线程：ForkJoinPool-1-worker-10
执行计算任务，用户数：3，结果是：6   执行此任务的线程：ForkJoinPool-1-worker-6
任务过大，用户数:12，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-15
执行计算任务，用户数：3，结果是：156   执行此任务的线程：ForkJoinPool-1-worker-13
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-15
任务过大，用户数:13，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-10
执行计算任务，用户数：3，结果是：15   执行此任务的线程：ForkJoinPool-1-worker-8
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-3
任务过大，用户数:7，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-6
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-1
执行计算任务，用户数：3，结果是：201   执行此任务的线程：ForkJoinPool-1-worker-2
执行计算任务，用户数：3，结果是：183   执行此任务的线程：ForkJoinPool-1-worker-4
执行计算任务，用户数：3，结果是：210   执行此任务的线程：ForkJoinPool-1-worker-5
执行计算任务，用户数：4，结果是：94   执行此任务的线程：ForkJoinPool-1-worker-9
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-13
执行计算任务，用户数：3，结果是：231   执行此任务的线程：ForkJoinPool-1-worker-1
执行计算任务，用户数：3，结果是：285   执行此任务的线程：ForkJoinPool-1-worker-6
执行计算任务，用户数：3，结果是：117   执行此任务的线程：ForkJoinPool-1-worker-3
执行计算任务，用户数：4，结果是：294   执行此任务的线程：ForkJoinPool-1-worker-8
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-10
执行计算任务，用户数：3，结果是：81   执行此任务的线程：ForkJoinPool-1-worker-15
执行计算任务，用户数：3，结果是：267   执行此任务的线程：ForkJoinPool-1-worker-10
执行计算任务，用户数：3，结果是：276   执行此任务的线程：ForkJoinPool-1-worker-4
任务过大，用户数:6，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-5
执行计算任务，用户数：3，结果是：249   执行此任务的线程：ForkJoinPool-1-worker-5
任务过大，用户数:7，切割的任务：执行此任务的线程：ForkJoinPool-1-worker-8
执行计算任务，用户数：3，结果是：126   执行此任务的线程：ForkJoinPool-1-worker-3
执行计算任务，用户数：4，结果是：194   执行此任务的线程：ForkJoinPool-1-worker-3
执行计算任务，用户数：3，结果是：240   执行此任务的线程：ForkJoinPool-1-worker-1
执行计算任务，用户数：3，结果是：99   执行此任务的线程：ForkJoinPool-1-worker-13
执行计算任务，用户数：4，结果是：394   执行此任务的线程：ForkJoinPool-1-worker-11
执行计算任务，用户数：3，结果是：90   执行此任务的线程：ForkJoinPool-1-worker-12
执行计算任务，用户数：3，结果是：135   执行此任务的线程：ForkJoinPool-1-worker-8
执行计算任务，用户数：3，结果是：258   执行此任务的线程：ForkJoinPool-1-worker-5
执行计算任务，用户数：3，结果是：108   执行此任务的线程：ForkJoinPool-1-worker-4
最终的计算结果：5050
```
### Fork/Join 框架的异常处理
`ForkJoinTask`在执行的时候可能会抛出异常，但是我们没办法在主线程里直接捕获异常，所以 `ForkJoinTask`提供了`isCompletedAbnormally()` 方法来检查任务是否已经抛出异常或已经被取消了，并且可以通过 ForkJoinTask 的 `getException`方法获取异常。使用如下代码
```
if(task.isCompletedAbnormally()) {
   System.out.println(task.getException());
}
```
`getException` 方法返回 `Throwable` 对象，如果任务被取消了则返回 `CancellationException`。如果任务没有完成或者没有抛出异常则返回 `null`。

### ForkJoinPool 使用 submit 与 invoke 提交的区别
- invoke 是同步执行，调用之后需要等待任务完成，才能执行后面的代码。
- submit 是异步执行，只有在 Future 调用 get 的时候会阻塞。

### 继承 RecursiveTask 与 RecursiveAction的区别？
- 继承 RecursiveTask：适用于有返回值的场景。
- 继承 RecursiveAction：适合于没有返回值的场景。

### 子任务调用 fork 与 invokeAll 的区别？
- fork：让子线程自己去完成任务，父线程监督子线程执行，浪费父线程。
- invokeAll：子父线程共同完成任务，可以更好的利用线程池。