# Library_management_java


import java.util.*;

// 🔹 Interface for Library operations
interface LibraryOperations {
    void addMembers();

    void addBooks();
}

interface Loanable {
    void issueBook() throws BookAlreadyIssuedException;
}

interface Displayable {
    void displayAvailableBooks();

    void displayRemainingBooks();
}

// 🔹 Custom exception
class BookAlreadyIssuedException extends Exception {
    public BookAlreadyIssuedException(String message) {
        super(message);
    }
}

// 🔹 Book class with toString
class Book {
    String title;
    boolean isIssued;

    Book(String title) {
        this.title = title;
        this.isIssued = false;
    }

    @Override
    public String toString() {
        return title + (isIssued ? " (Issued)" : "");
    }
}

// 🔹 Main library class
class Library implements LibraryOperations, Loanable, Displayable {
    Scanner sc = new Scanner(System.in);
    String[] facultyMembers;
    String[] studentMembers;
    Book[] books;

    // Map to track issued books per member
    Map<String, Integer> booksIssuedCount = new HashMap<>();

    @Override
    public void addMembers() {
        int facultySize = getValidatedInt("Enter number of faculty members: ");
        int studentSize = getValidatedInt("Enter number of students: ");

        facultyMembers = new String[facultySize];
        studentMembers = new String[studentSize];

        for (int i = 0; i < facultySize; i++) {
            System.out.print("Enter faculty name " + (i + 1) + ": ");
            facultyMembers[i] = sc.nextLine();
        }

        for (int i = 0; i < studentSize; i++) {
            System.out.print("Enter student name " + (i + 1) + ": ");
            studentMembers[i] = sc.nextLine();
        }
    }

    @Override
    public void addBooks() {
        int bookSize = getValidatedInt("Enter number of books in library: ");
        books = new Book[bookSize];

        for (int i = 0; i < bookSize; i++) {
            System.out.print("Enter name of book " + (i + 1) + ": ");
            books[i] = new Book(sc.nextLine());
        }
    }

    @Override
    public void issueBook() throws BookAlreadyIssuedException {
        if (!dataInitialized()) {
            System.out.println("⚠ Please add members and books first.");
            return;
        }

        System.out.print("Enter member name to issue book: ");
        String name = sc.nextLine();

        String type = getMemberType(name);
        if (type == null) {
            System.out.println(" Member not found.");
            return;
        }

        int maxLimit = type.equals("Faculty") ? 10 : 5;
        int alreadyIssued = booksIssuedCount.getOrDefault(name, 0);
        int available = countAvailableBooks();

        if (available == 0) {
            System.out.println("📚 No books are currently available.");
            return;
        }

        int issueCount = getValidatedInt("Enter number of books to issue (Max " + maxLimit + "): ");
        if (issueCount + alreadyIssued > maxLimit) {
            System.out.println(type + " can issue only " + (maxLimit - alreadyIssued) + " more books.");
            issueCount = maxLimit - alreadyIssued;
        }

        if (issueCount > available) {
            System.out.println("Only " + available + " books available. Issuing " + available + " only.");
            issueCount = available;
        }

        displayAvailableBooks();

        int issuedNow = 0;

        while (issuedNow < issueCount) {
            int bookIndex = getValidatedInt(
                    "Enter book number to issue (" + (issuedNow + 1) + " of " + issueCount + "): ") - 1;

            if (bookIndex >= 0 && bookIndex < books.length) {
                if (!books[bookIndex].isIssued) {
                    books[bookIndex].isIssued = true;
                    System.out.println("Issued: \"" + books[bookIndex].title + "\" to " + name);
                    issuedNow++;
                } else {
                    throw new BookAlreadyIssuedException("Book already issued: " + books[bookIndex].title);
                }
            } else {
                System.out.println("Invalid selection. Try again.");
            }
        }

        booksIssuedCount.put(name, alreadyIssued + issuedNow);
        displayRemainingBooks();
    }

    @Override
    public void displayAvailableBooks() {
        System.out.println("\n Available Books:");
        for (int i = 0; i < books.length; i++) {
            if (!books[i].isIssued) {
                System.out.println((i + 1) + ". " + books[i]);
            }
        }
    }

    @Override
    public void displayRemainingBooks() {
        System.out.println("\nRemaining Books:");
        boolean anyLeft = false;
        for (Book book : books) {
            if (!book.isIssued) {
                System.out.println("- " + book);
                anyLeft = true;
            }
        }
        if (!anyLeft) {
            System.out.println("All books have been issued.");
        }
    }

    // Utility: safe int input
    private int getValidatedInt(String prompt) {
        int value;
        while (true) {
            try {
                System.out.print(prompt);
                value = sc.nextInt();
                sc.nextLine(); // clear buffer
                if (value < 0) {
                    System.out.println(" Enter a positive number.");
                } else {
                    return value;
                }
            } catch (InputMismatchException e) {
                System.out.println("Invalid input. Please enter a number.");
                sc.nextLine();
            }
        }
    }

    // Check if all necessary data is added
    private boolean dataInitialized() {
        return facultyMembers != null && studentMembers != null && books != null;
    }

    // Determine member type
    private String getMemberType(String name) {
        for (String faculty : facultyMembers) {
            if (faculty.equalsIgnoreCase(name))
                return "Faculty";
        }
        for (String student : studentMembers) {
            if (student.equalsIgnoreCase(name))
                return "Student";
        }
        return null;
    }

    // Count how many books can still be issued
    private int countAvailableBooks() {
        int count = 0;
        for (Book book : books) {
            if (!book.isIssued)
                count++;
        }
        return count;
    }
}

// 🔹 Entry point
public class Library_Management {
    public static void main(String[] args) {
        Library library = new Library();

        try {
            library.addMembers();
            library.addBooks();
            library.issueBook();
        } catch (BookAlreadyIssuedException e) {
            System.out.println(e.getMessage());
        } catch (Exception e) {
            System.out.println("Unexpected error: " + e.getMessage());
        }
    }
}
