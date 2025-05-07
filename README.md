# Database


-- 1. Role Table
CREATE TABLE Role (
    role_id INT PRIMARY KEY AUTO_INCREMENT,
    role_name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 2. Permission Table
CREATE TABLE Permission (
    permission_id INT PRIMARY KEY AUTO_INCREMENT,
    permission_name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 3. RolePermission Table
CREATE TABLE RolePermission (
    role_id INT,
    permission_id INT,
    granted_by INT,
    granted_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (role_id, permission_id),
    FOREIGN KEY (role_id) REFERENCES Role(role_id),
    FOREIGN KEY (permission_id) REFERENCES Permission(permission_id),
    FOREIGN KEY (granted_by) REFERENCES User(user_id)
);

-- 4. User Table
CREATE TABLE User (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE,
    password VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    address TEXT,
    profile_picture VARCHAR(255),
    status ENUM('Active', 'Inactive', 'Suspended') DEFAULT 'Active',
    role_id INT,
    manager_id INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (role_id) REFERENCES Role(role_id),
    FOREIGN KEY (manager_id) REFERENCES User(user_id)
);

-- 5. Skill Table
CREATE TABLE Skill (
    skill_id INT PRIMARY KEY AUTO_INCREMENT,
    skill_name VARCHAR(100) NOT NULL,
    skill_description VARCHAR(255),
    category VARCHAR(100),
    level ENUM('Beginner', 'Intermediate', 'Advanced') DEFAULT 'Beginner',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- 6. Task Table
CREATE TABLE Task (
    task_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(100),
    description TEXT,
    priority ENUM('Low', 'Medium', 'High') DEFAULT 'Medium',
    status ENUM('Assigned', 'In Progress', 'Completed', 'Blocked') DEFAULT 'Assigned',
    estimated_hours INT,
    actual_hours INT,
    required_skill_id INT,
    assigned_to INT,
    deadline DATE,
    start_date DATE,
    end_date DATE,
    created_by INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (required_skill_id) REFERENCES Skill(skill_id),
    FOREIGN KEY (assigned_to) REFERENCES User(user_id),
    FOREIGN KEY (created_by) REFERENCES User(user_id)
);

-- 7. TaskUpdate Table
CREATE TABLE TaskUpdate (
    update_id INT PRIMARY KEY AUTO_INCREMENT,
    task_id INT,
    comment TEXT,
    status ENUM('In Progress', 'Completed', 'Blocked') NOT NULL,
    updated_by INT,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (task_id) REFERENCES Task(task_id),
    FOREIGN KEY (updated_by) REFERENCES User(user_id)
);

-- 8. Report Table
CREATE TABLE Report (
    report_id INT PRIMARY KEY AUTO_INCREMENT,
    task_id INT,
    status ENUM('Completed', 'Pending', 'Overdue') NOT NULL,
    progress_percent INT CHECK (progress_percent BETWEEN 0 AND 100),
    remarks TEXT,
    report_date DATE,
    created_by INT,
    FOREIGN KEY (task_id) REFERENCES Task(task_id),
    FOREIGN KEY (created_by) REFERENCES User(user_id)
);

-- 9. Notification Table
CREATE TABLE Notification (
    notification_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    message TEXT,
    type ENUM('Alert', 'Info', 'Reminder') DEFAULT 'Info',
    is_read BOOLEAN DEFAULT FALSE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES User(user_id)
);

-- 10. SkillHistory Table (certification_link removed)
CREATE TABLE SkillHistory (
    history_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    skill_id INT,
    acquired_on DATE,
    remarks TEXT,
    FOREIGN KEY (user_id) REFERENCES User(user_id),
    FOREIGN KEY (skill_id) REFERENCES Skill(skill_id)
);

-- 11. TeamGroup Table
CREATE TABLE TeamGroup (
    team_id INT PRIMARY KEY AUTO_INCREMENT,
    team_name VARCHAR(100),
    project_name VARCHAR(100),
    created_by INT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (created_by) REFERENCES User(user_id)
);

-- 12. TeamMember Table
CREATE TABLE TeamMember (
    user_id INT,
    team_id INT,
    skill_id INT,
    task_id INT,
    joined_on DATE,
    role_in_team VARCHAR(100),
    PRIMARY KEY (user_id, team_id),
    FOREIGN KEY (user_id) REFERENCES User(user_id),
    FOREIGN KEY (team_id) REFERENCES TeamGroup(team_id),
    FOREIGN KEY (skill_id) REFERENCES Skill(skill_id),
    FOREIGN KEY (task_id) REFERENCES Task(task_id)
);

-- 13. TaskRequest Table
CREATE TABLE TaskRequest (
    request_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    task_id INT,
    request_message TEXT,
    request_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    status ENUM('Requested', 'Approved', 'Rejected') DEFAULT 'Requested',
    responded_by INT,
    responded_at DATETIME,
    FOREIGN KEY (user_id) REFERENCES User(user_id),
    FOREIGN KEY (task_id) REFERENCES Task(task_id),
    FOREIGN KEY (responded_by) REFERENCES User(user_id)
);

-- 14. UserSkill Table
CREATE TABLE UserSkill (
    user_id INT,
    skill_id INT,
    proficiency_level ENUM('Beginner', 'Intermediate', 'Advanced') DEFAULT 'Beginner',
    last_used DATE,
    PRIMARY KEY (user_id, skill_id),
    FOREIGN KEY (user_id) REFERENCES User(user_id),
    FOREIGN KEY (skill_id) REFERENCES Skill(skill_id)
);

-- 15. TaskSkill Table
CREATE TABLE TaskSkill (
    task_id INT,
    skill_id INT,
    importance_level ENUM('Low', 'Medium', 'High') DEFAULT 'Medium',
    PRIMARY KEY (task_id, skill_id),
    FOREIGN KEY (task_id) REFERENCES Task(task_id),
    FOREIGN KEY (skill_id) REFERENCES Skill(skill_id)
);






-- Insert Roles
INSERT INTO Role (role_name, description) VALUES
('Admin', 'Administrator with full access'),
('Manager', 'Manages tasks and teams'),
('Developer', 'Executes assigned tasks');

-- Insert Permissions
INSERT INTO Permission (permission_name, description) VALUES
('Assign Task', 'Permission to assign tasks to users'),
('Approve Task Request', 'Can approve or reject task requests'),
('Create Report', 'Can create task progress reports');

-- Insert Users
INSERT INTO User (name, email, password, phone, address, role_id) VALUES
('Alice Admin', 'alice@example.com', 'hashed_pwd_1', '1234567890', '123 Admin St', 1),
('Bob Manager', 'bob@example.com', 'hashed_pwd_2', '2345678901', '456 Manager Rd', 2),
('Charlie Dev', 'charlie@example.com', 'hashed_pwd_3', '3456789012', '789 Dev Ln', 3);

-- Insert RolePermissions
INSERT INTO RolePermission (role_id, permission_id, granted_by) VALUES
(1, 1, 1),
(1, 2, 1),
(1, 3, 1),
(2, 1, 1),
(2, 2, 1),
(3, 3, 1);

-- Insert Skills
INSERT INTO Skill (skill_name, skill_description, category, level) VALUES
('Python', 'Python programming language', 'Programming', 'Advanced'),
('SQL', 'Structured Query Language', 'Database', 'Intermediate'),
('Team Communication', 'Effective communication in teams', 'Soft Skill', 'Intermediate');

-- Insert Tasks
INSERT INTO Task (title, description, priority, status, estimated_hours, actual_hours, required_skill_id, assigned_to, deadline, start_date, end_date, created_by) VALUES
('Build API', 'Develop backend API using Python', 'High', 'Assigned', 10, NULL, 1, 3, '2025-05-20', '2025-05-10', NULL, 2),
('Design DB', 'Design and implement database schema', 'Medium', 'Assigned', 8, NULL, 2, 3, '2025-05-22', '2025-05-11', NULL, 2);

-- Insert Task Updates
INSERT INTO TaskUpdate (task_id, comment, status, updated_by) VALUES
(1, 'Started working on API endpoints', 'In Progress', 3),
(2, 'Schema draft prepared', 'In Progress', 3);

-- Insert Reports
INSERT INTO Report (task_id, status, progress_percent, remarks, report_date, created_by) VALUES
(1, 'Pending', 30, 'Initial development done', '2025-05-12', 3),
(2, 'Pending', 50, 'ER diagram created', '2025-05-13', 3);

-- Insert Notifications
INSERT INTO Notification (user_id, message, type) VALUES
(3, 'New task assigned: Build API', 'Alert'),
(3, 'Reminder: Report due tomorrow', 'Reminder');

-- Insert SkillHistory (certification_link removed)
INSERT INTO SkillHistory (user_id, skill_id, acquired_on, remarks) VALUES
(3, 1, '2024-10-01', 'Learned via online course'),
(3, 2, '2024-09-15', 'Practiced in project');

-- Insert TeamGroup
INSERT INTO TeamGroup (team_name, project_name, created_by) VALUES
('Backend Team', 'Project Alpha', 2),
('DB Team', 'Project Beta', 2);

-- Insert TeamMember
INSERT INTO TeamMember (user_id, team_id, skill_id, task_id, joined_on, role_in_team) VALUES
(3, 1, 1, 1, '2025-05-09', 'Backend Developer'),
(3, 2, 2, 2, '2025-05-10', 'DB Engineer');

-- Insert TaskRequest
INSERT INTO TaskRequest (user_id, task_id, request_message, status, responded_by, responded_at) VALUES
(3, 1, 'I would like to work on this API task', 'Approved', 2, '2025-05-09 09:00:00');
// --- Entity and DTO Classes for Task, TaskUpdate, Report, Notification, SkillHistory, TeamGroup, TeamMember, TaskRequest, UserSkill, TaskSkill ---

// Entity and DTO for Task @Entity public class Task { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer taskId; private String title; private String description; @Enumerated(EnumType.STRING) private Priority priority; @Enumerated(EnumType.STRING) private Status status; private Integer estimatedHours; private Integer actualHours; @ManyToOne private Skill requiredSkill; @ManyToOne private User assignedTo; @ManyToOne private User createdBy; private LocalDate deadline; private LocalDate startDate; private LocalDate endDate; private LocalDateTime createdAt; private LocalDateTime updatedAt; }

@Data public class TaskDTO { private Integer taskId; private String title; private String description; private String priority; private String status; private Integer estimatedHours; private Integer actualHours; private Integer requiredSkillId; private Integer assignedToId; private Integer createdById; private LocalDate deadline; private LocalDate startDate; private LocalDate endDate; }

// Entity and DTO for TaskUpdate @Entity public class TaskUpdate { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer updateId; @ManyToOne private Task task; private String comment; @Enumerated(EnumType.STRING) private TaskStatus status; @ManyToOne private User updatedBy; private LocalDateTime updatedAt; }

@Data public class TaskUpdateDTO { private Integer updateId; private Integer taskId; private String comment; private String status; private Integer updatedById; }

// Entity and DTO for Report @Entity public class Report { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer reportId; @ManyToOne private Task task; @Enumerated(EnumType.STRING) private ReportStatus status; private Integer progressPercent; private String remarks; private LocalDate reportDate; @ManyToOne private User createdBy; }

@Data public class ReportDTO { private Integer reportId; private Integer taskId; private String status; private Integer progressPercent; private String remarks; private LocalDate reportDate; private Integer createdById; }

// Entity and DTO for Notification @Entity public class Notification { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer notificationId; @ManyToOne private User user; private String message; @Enumerated(EnumType.STRING) private NotificationType type; private Boolean isRead; private LocalDateTime createdAt; }

@Data public class NotificationDTO { private Integer notificationId; private Integer userId; private String message; private String type; private Boolean isRead; private LocalDateTime createdAt; }

// Entity and DTO for SkillHistory @Entity public class SkillHistory { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer historyId; @ManyToOne private User user; @ManyToOne private Skill skill; private LocalDate acquiredOn; private String remarks; }

@Data public class SkillHistoryDTO { private Integer historyId; private Integer userId; private Integer skillId; private LocalDate acquiredOn; private String remarks; }

// Entity and DTO for TeamGroup @Entity public class TeamGroup { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer teamId; private String teamName; private String projectName; @ManyToOne private User createdBy; private LocalDateTime createdAt; }

@Data public class TeamGroupDTO { private Integer teamId; private String teamName; private String projectName; private Integer createdById; private LocalDateTime createdAt; }

// Entity and DTO for TeamMember @Entity @IdClass(TeamMemberId.class) public class TeamMember { @Id private Integer userId; @Id private Integer teamId; @ManyToOne private Skill skill; @ManyToOne private Task task; private LocalDate joinedOn; private String roleInTeam; }

@Data public class TeamMemberDTO { private Integer userId; private Integer teamId; private Integer skillId; private Integer taskId; private LocalDate joinedOn; private String roleInTeam; }

// Entity and DTO for TaskRequest @Entity public class TaskRequest { @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Integer requestId; @ManyToOne private User user; @ManyToOne private Task task; private String requestMessage; private LocalDateTime requestDate; @Enumerated(EnumType.STRING) private RequestStatus status; @ManyToOne private User respondedBy; private LocalDateTime respondedAt; }

@Data public class TaskRequestDTO { private Integer requestId; private Integer userId; private Integer taskId; private String requestMessage; private LocalDateTime requestDate; private String status; private Integer respondedById; private LocalDateTime respondedAt; }

// Entity and DTO for UserSkill @Entity @IdClass(UserSkillId.class) public class UserSkill { @Id private Integer userId; @Id private Integer skillId; @Enumerated(EnumType.STRING) private ProficiencyLevel proficiencyLevel; private LocalDate lastUsed; }

@Data public class UserSkillDTO { private Integer userId; private Integer skillId; private String proficiencyLevel; private LocalDate lastUsed; }

// Entity and DTO for TaskSkill @Entity @IdClass(TaskSkillId.class) public class TaskSkill { @Id private Integer taskId; @Id private Integer skillId; @Enumerated(EnumType.STRING) private ImportanceLevel importanceLevel; }

@Data public class TaskSkillDTO { private Integer taskId; private Integer skillId; private String importanceLevel; }



-- Insert UserSkill
INSERT INTO UserSkill (user_id, skill_id, proficiency_level, last_used) VALUES
(3, 1, 'Advanced', '2025-05-07'),
(3, 2, 'Intermediate', '2025-05-06');

-- Insert TaskSkill
INSERT INTO TaskSkill (task_id, skill_id, importance_level) VALUES
(1, 1, 'High'),
(2, 2, 'Medium');
