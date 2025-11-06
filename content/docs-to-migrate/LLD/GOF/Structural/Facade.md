---
# 786
weight: 600
---
# GOF Design Patterns : Façade

## Introduction

The Façade Design Pattern is one of the GOF Design Patterns that falls under structural design patterns. This pattern is used to hide complex business logic behind and expose it via simple interfaces. So, the client classes do not need to worry about the logic that runs behind the scenes.

We can define the Façade Design Pattern as,

> A design pattern that defines a higher-level unified interface to a set of interfaces in a subsystem that makes the subsystem easier to use

## Problem

Let’s try to understand this with a simple example. Assume we have an application that users can use to share invoice files with other users. The invoice sharing method looks like this.

```java
public class FacadeDemo {

    SessionManager sessionManager;
    UserRepository userRepository;
    ValidationHelper validationHelper;
    FileManager fileManager;
    Scanner scanner;
    S3Manager s3Manager;
    FileShareRepository fileShareRepository;
    EmailManager emailManager;
    NotificationManager notificationManager;

    public FacadeDemo(SessionManager sessionManager, UserRepository userRepository, ValidationHelper validationHelper,
        FileManager fileManager, Scanner scanner, S3Manager s3Manager, FileShareRepository fileShareRepository,
        EmailManager emailManager, NotificationManager notificationManager) {
        super();
        this.sessionManager = sessionManager;
        this.userRepository = userRepository;
        this.validationHelper = validationHelper;
        this.fileManager = fileManager;
        this.scanner = scanner;
        this.s3Manager = s3Manager;
        this.fileShareRepository = fileShareRepository;
        this.emailManager = emailManager;
        this.notificationManager = notificationManager;
    }

    public ResponseEntity shareFileWithUser(HttpServletRequest request, MultipartFile file, UUID receiverId)
    throws CustomException {

        // Simplified Code Used For Demonstration Purpose Only.

        // Get Current Logged In User
        UUID loggedInUserId = sessionManager.getLoggedInUser(request);

        // Fetch Users
        User sender = userRepository.findUserById(loggedInUserId);
        User receiver = userRepository.findUserById(receiverId);

        // Validate Users
        validationHelper.validateSender(sender);
        validationHelper.validateReceiver(receiver);

        // Validate Relationship
        validationHelper.validateRelationship(sender, receiver);

        // Validate File & Scan
        validationHelper.validateFile(file);
        scanner.scanFile(file);

        // Extract MetaData
        FileMeta metaData = fileManager.extractMetaData(file);

        // Upload File To S3
        String path = fileManager.calculateUploadPath(sender, receiver, file);
        s3Manager.uploadFile(path, file);

        // Save to DB
        FileShare fileShare = fileShareRepository.saveFileShareInfo(sender, receiver, metaData, path);

        // Send Email
        emailManager.notifyAndUpdateStaus(fileShare); // ASYNC

        // Send Push Notification
        notificationManager.sendPushNotificationAndUpdateStaus(fileShare); // ASYNC

        // Generate Response
        return new ResponseEntity("Upload Success", true);

    }

}
```

As we can see, the code above has many issues with it.

- **Readability**: The method is lengthy, multiple interactions are compiled into one single method, and readability is low
- **Tightly Coupled**: The method is tightly coupled with multiple services such as `sessionManager`, `userRepository`, `validationHelper`, `fileManager`, `scanner`, `s3Manager`, `fileShareRepository`, `emailManager`, `notificationManager`.
- **Hard to Maintain**: Since the method is tightly coupled with a large number of other services, it will become hard to maintain over time.
- **Hard to Test**: Since multiple interactions are compiled into one single method, there are large number of possible flows that need to be tested for this single method. Testing of this method requires heavy mocking and stubbing, which will become more complex.
- **Reusability**: Assume we have another method, which is used to share a daily summary, but that does not require email or push notification. In such a case, we need to repeat the whole logic without the notification sending part. This will lead to code duplication issues in static analyzers like SonarQube. This will also violates the DRY (Don't Repeat Yourself) principle.
- **Refactoring**: Also in such repeated cases, assume we decided to use Azure Blob instead of AWS S3, we will have to refactor all occurrences again, and the code will become error-prone.

## Solution

As the title of the article says, we can solve these issues by applying the Façade pattern to the above code. Let's see how it resolves these issues.

FacadeDemo.java
```java
public class FacadeDemo {

	
	UserManagerFacade userManagerFacade;
	FileManagerFacade fileManagerFacade;
	NotificationManagerFacade notificationManagerFacade;

	public FacadeDemo(UserManagerFacade userManagerFacade, FileManagerFacade fileManagerFacade,
			NotificationManagerFacade notificationManagerFacade) {
		super();
		this.userManagerFacade = userManagerFacade;
		this.fileManagerFacade = fileManagerFacade;
		this.notificationManagerFacade = notificationManagerFacade;
	}

	public ResponseEntity shareFileWithUser(HttpServletRequest request, MultipartFile file, UUID receiverId)
			throws CustomException {
		// Simplified Code Used For Demonstration Purpose Only.

		// Get Current Logged In User
		User sender = userManagerFacade.getLoggedInUser(request);
		
		// Validate and Get Receiver
		User receiver = userManagerFacade.validateAndGetReceiver(sender, receiverId);

		// Validate, Process & Upload File
		FileShare fileShare = fileManagerFacade.validateAndShareFile(sender, receiver, file);

		// Send Email & Push Notification
		notificationManagerFacade.notifyUser(fileShare,true,true);

		// Generate Response
		return new ResponseEntity("Upload Success", true);
	}
}
```

UserManagerFacade.java
```java
public class UserManagerFacade {
	
	SessionManager sessionManager;
	UserRepository userRepository;
	ValidationHelper validationHelper;
	
	public UserManagerFacade(SessionManager sessionManager, UserRepository userRepository, ValidationHelper validationHelper) {
		super();
		this.sessionManager = sessionManager;
		this.userRepository = userRepository;
		this.validationHelper = validationHelper;
	}

	public User getLoggedInUser(HttpServletRequest request) {
		UUID loggedInUserId = sessionManager.getLoggedInUser(request);
		User sender = userRepository.findUserById(loggedInUserId);
		validationHelper.validateSender(sender);
		return sender;
	}

	public User validateAndGetReceiver(User sender, UUID receiverId) {
		User receiver = userRepository.findUserById(receiverId);
		validationHelper.validateReceiver(receiver);
		validationHelper.validateRelationship(sender, receiver);
		return receiver;
	}
}
```

FileManagerFacade.java
```java
public class FileManagerFacade {
	
	ValidationHelper validationHelper;
	FileManager fileManager;
	Scanner scanner;
	S3Manager s3Manager;
	FileShareRepository fileShareRepository;
	
	public FileManagerFacade(ValidationHelper validationHelper, FileManager fileManager, Scanner scanner,
			S3Manager s3Manager, FileShareRepository fileShareRepository) {
		super();
		this.validationHelper = validationHelper;
		this.fileManager = fileManager;
		this.scanner = scanner;
		this.s3Manager = s3Manager;
		this.fileShareRepository = fileShareRepository;
	}

	public FileShare validateAndShareFile(User sender, User receiver, MultipartFile file) {
		validationHelper.validateFile(file);
		scanner.scanFile(file);
		FileMeta metaData = fileManager.extractMetaData(file);
		String path = fileManager.calculateUploadPath(sender, receiver, file);
		s3Manager.uploadFile(path, file);
		FileShare fileShare = fileShareRepository.saveFileShareInfo(sender, receiver, metaData, path);
		return fileShare;
	}
}
```

NotificationManagerFacade.java
```java
public class NotificationManagerFacade {

	EmailManager emailManager;
	NotificationManager notificationManager;
	
	public NotificationManagerFacade(EmailManager emailManager, NotificationManager notificationManager) {
		super();
		this.emailManager = emailManager;
		this.notificationManager = notificationManager;
	}

	public void notifyUser(FileShare fileShare, boolean sendEmail, boolean sendPushNotification) {
		if(sendEmail) emailManager.notifyAndUpdateStaus(fileShare); // ASYNC
		if(sendPushNotification) notificationManager.sendPushNotificationAndUpdateStaus(fileShare); // ASYNC
	}

}
```

Now let's re-evaluate our code.

As we can see, the client classes do not need to worry about the underlying logic of `UserManagerFacade`, `NotificationManagerFacade`, `FileManagerFacade`. Since the coupling between classes is loose and each class is responsible for only a specific scope (separation of concerns), it becomes easy to maintain and test. This also improves the readability of the code. 

When it comes to refactoring and reusability, we just need to refactor and reuse relevant facade methods only, which makes things a lot easier. And the client classes can use these facade methods repeatedly without worrying about the underlying logic.

As we can see, applying the Façade pattern solves multiple problems, that we may face in the long run, at an early stage.

## Class Diagrams

To get a better understanding, let's try to look at how the class diagrams look before and after applying the Façade pattern.

### Before
![](/images/facade-before.png)

### After
![](/images/facade-after.png)

As we can clearly see, the Façade Pattern helps us to hide complex logic behind and make the interaction of the client class loosely coupled, cleaner, and more maintainable.

That's all for Façade Pattern for now. Hope this article was helpful.\
Thank you !\
Happy Coding 🙌