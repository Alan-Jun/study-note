# AtomicMarkableReference

和 `AtomicStampedReference` 基本相同不过 `final int stamp` 变成了 `final boolean mark`

# 基本结构代码

```java
public class AtomicMarkableReference<V> {
    //这里几乎和AtomicStampedReference相同，只是从 final int stamp 变成了 final boolean mark
    private static class Pair<T> {
        final T reference;
        final boolean mark;
        private Pair(T reference, boolean mark) {
            this.reference = reference;
            this.mark = mark;
        }
        static <T> Pair<T> of(T reference, boolean mark) {
            return new Pair<T>(reference, mark);
        }
    }

    private volatile Pair<V> pair;

    /**
     * Creates a new {@code AtomicMarkableReference} with the given
     * initial values.
     *
     * @param initialRef the initial reference
     * @param initialMark the initial mark
     */
    public AtomicMarkableReference(V initialRef, boolean initialMark) {
        pair = Pair.of(initialRef, initialMark);
    }
}
```

