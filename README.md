import java.util.*;

class BookNotFoundException extends Exception {
    public BookNotFoundException(String message) {
        super(message);
    }
}

class Book {
    int id;
    String title;
    boolean isIssued;

    public Book(int id, String title) {
        this.id = id;
        this.title = title;
        this.isIssued = false;
    }
}

class Library {
    private final Map<Integer, Book> books = Collections.synchronizedMap(new HashMap<>());

    public Library() {
        books.put(1, new Book(1, "Java Programming"));
        books.put(2, new Book(2, "Data Structures"));
        books.put(3, new Book(3, "Operating Systems"));
    }

    public synchronized void issueBook(int bookId) throws BookNotFoundException {
        Book book = books.get(bookId);
        if (book == null) {
            throw new BookNotFoundException("Book with ID " + bookId + " not found.");
        }
        if (book.isIssued) {
            System.out.println("Book '" + book.title + "' is already issued.");
        } else {
            book.isIssued = true;
            System.out.println("Book '" + book.title + "' issued successfully.");
        }
    }

    public synchronized void returnBook(int bookId) throws BookNotFoundException {
        Book book = books.get(bookId);
        if (book == null) {
            throw new BookNotFoundException("Book with ID " + bookId + " not found.");
        }
        if (!book.isIssued) {
            System.out.println("Book '" + book.title + "' was not issued.");
        } else {
            book.isIssued = false;
            System.out.println("Book '" + book.title + "' returned successfully.");
        }
    }
}

class LibraryTask implements Runnable {
    private final Library library;
    private final int bookId;
    private final boolean issue;

    public LibraryTask(Library library, int bookId, boolean issue) {
        this.library = library;
        this.bookId = bookId;
        this.issue = issue;
    }

    @Override
    public void run() {
        try {
            if (issue) {
                library.issueBook(bookId);
            } else {
                library.returnBook(bookId);
            }
        } catch (BookNotFoundException e) {
            System.out.println(e.getMessage());
        }
    }
}

public class LibraryManagementSystem {
    public static void main(String[] args) {
        Library library = new Library();

        Thread t1 = new Thread(new LibraryTask(library, 1, true));
        Thread t2 = new Thread(new LibraryTask(library, 1, false));
        Thread t3 = new Thread(new LibraryTask(library, 2, true));
        Thread t4 = new Thread(new LibraryTask(library, 4, true));

        t1.start();
        t2.start();
        t3.start();
        t4.start();

        try {
            t1.join();
            t2.join();
            t3.join();
            t4.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("Library operations completed.");
    }
}
