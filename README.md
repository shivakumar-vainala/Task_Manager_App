package TaskManagerApp;

import java.io.*;
import java.util.ArrayList;
import java.util.List;

interface Noteable {
    void createNote();
    void editNote();
}

class AuthenticationFailedException extends Exception {
    public AuthenticationFailedException(String message) {
        super(message);
    }
}

class User {
    private String username;
    private String password;

    public User(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public User() {
        this.username = "xyz";
        this.password = "abcd";
    }

    public boolean authenticateUser(String inputPassword) throws AuthenticationFailedException {
        if (!this.password.equals(inputPassword)) {
            throw new AuthenticationFailedException("Authentication failed.");
        }
        return true;
    }
}

class Task {
    protected String taskName;
    protected int priority;

    public Task(String taskName, int priority) {
        this.taskName = taskName;
        this.priority = priority;
    }
}

class ImportantTask extends Task {
    private boolean isUrgent;

    public ImportantTask(String taskName, int priority, boolean isUrgent) {
        super(taskName, priority);
        this.isUrgent = isUrgent;
    }
}

class SimpleNote implements Noteable {
    private String title;
    private String content;

    public SimpleNote(String title, String content) {
        this.title = title;
        this.content = content;
    }

    @Override
    public void createNote() {
        System.out.println("Note created: " + title);
    }

    @Override
    public void editNote() {
        System.out.println("Editing note: " + title);
    }
}

final class PasswordManager {
    private static List<String> savedPasswords = new ArrayList<>();

    public static void addPassword(String password) {
        savedPasswords.add(password);
    }
}

public class TaskManagerApp {
    public static void main(String[] args) {
        User user = new User("johnDoe", "password123");

        int retryCount = 3;
        boolean isAuthenticated = false;

        while (retryCount > 0 && !isAuthenticated) {
            try {
                isAuthenticated = user.authenticateUser("wrongPassword");
                System.out.println("User authenticated successfully.");
            } catch (AuthenticationFailedException e) {
                System.err.println(e.getMessage() + " Attempts remaining: " + (--retryCount));
                if (retryCount == 0) {
                    System.err.println("Access denied. Exiting application.");
                    System.exit(1);
                }
            }
        }

        ImportantTask task = new ImportantTask("Task 1", 2, true);

        if (task.priority < 1 || task.priority > 5) {
            System.err.println("Task priority must be between 1 and 5.");
        } else {
            System.out.println("Task created with valid priority.");
        }

        Noteable note = new SimpleNote("Important Note", "This is an important note.");
        note.createNote();

        PasswordManager.addPassword("mySecurePassword");

        Thread taskThread = new Thread(() -> {
            System.out.println("Task management thread running...");
        });
        taskThread.start();

        try {
            BufferedWriter writer = new BufferedWriter(new FileWriter("task.txt"));
            writer.write(task.taskName);
            writer.newLine();
            writer.write(String.valueOf(task.priority));
            writer.close();

            BufferedReader reader = new BufferedReader(new FileReader("task.txt"));
            String readTaskName = reader.readLine();
            int readPriority = Integer.parseInt(reader.readLine());
            reader.close();

            System.out.println("Task Read from File: " + readTaskName + ", Priority: " + readPriority);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

