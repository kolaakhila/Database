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


-- Insert UserSkill
INSERT INTO UserSkill (user_id, skill_id, proficiency_level, last_used) VALUES
(3, 1, 'Advanced', '2025-05-07'),
(3, 2, 'Intermediate', '2025-05-06');

-- Insert TaskSkill
INSERT INTO TaskSkill (task_id, skill_id, importance_level) VALUES
(1, 1, 'High'),
(2, 2, 'Medium');
Based on your database schema, here are the Entity and DTO classes in Java (using Spring Boot and Lombok) for the following tables:
// 1. Role
public class RoleDTO { 
    private int roleId; 
    private String roleName; 
    private String description; 
}

@Entity
public class Role {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int roleId;
    private String roleName;
    private String description;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @OneToMany(mappedBy = "role")
    private List<User> users;
}

// 2. Permission
public class PermissionDTO { 
    private int permissionId; 
    private String permissionName; 
    private String description; 
}

@Entity
public class Permission {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int permissionId;
    private String permissionName;
    private String description;
    private LocalDateTime createdAt;

    @ManyToMany(mappedBy = "permissions")
    private List<Role> roles;
}

// 3. RolePermission
public class RolePermissionDTO { 
    private int roleId; 
    private int permissionId; 
    private int grantedBy; 
}

@Entity
@IdClass(RolePermissionId.class)
public class RolePermission {
    @Id
    private int roleId;
    
    @Id
    private int permissionId;

    private int grantedBy;
    private LocalDateTime grantedAt;

    @ManyToOne
    @JoinColumn(name = "roleId", insertable = false, updatable = false)
    private Role role;

    @ManyToOne
    @JoinColumn(name = "permissionId", insertable = false, updatable = false)
    private Permission permission;
}

// 4. User
public class UserDTO { 
    private int userId; 
    private String name; 
    private String email; 
    private String phone; 
    private String address; 
    private String status; 
    private int roleId; 
    private int managerId; 
}

@Entity
public class User {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int userId;
    private String name;
    private String email;
    private String password;
    private String phone;
    private String address;
    private String profilePicture;
    private String status;
    private int roleId;
    private int managerId;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @ManyToOne
    @JoinColumn(name = "roleId", insertable = false, updatable = false)
    private Role role;

    @ManyToOne
    @JoinColumn(name = "managerId", insertable = false, updatable = false)
    private User manager;

    @OneToMany(mappedBy = "manager")
    private List<User> managedUsers;
}

// 5. Skill
public class SkillDTO { 
    private int skillId; 
    private String skillName; 
    private String skillDescription; 
    private String category; 
    private String level; 
}

@Entity
public class Skill {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int skillId;
    private String skillName;
    private String skillDescription;
    private String category;
    private String level;
    private LocalDateTime createdAt;

    @OneToMany(mappedBy = "skill")
    private List<UserSkill> userSkills;
    @OneToMany(mappedBy = "skill")
    private List<TaskSkill> taskSkills;
}

// 6. Task
public class TaskDTO { 
    private int taskId; 
    private String title; 
    private String description; 
    private String priority; 
    private String status; 
    private int requiredSkillId; 
    private int assignedTo; 
    private Date deadline; 
    private Date startDate; 
    private Date endDate; 
}

@Entity
public class Task {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int taskId;
    private String title;
    private String description;
    private String priority;
    private String status;
    private int estimatedHours;
    private int actualHours;
    private int requiredSkillId;
    private int assignedTo;
    private Date deadline;
    private Date startDate;
    private Date endDate;
    private int createdBy;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @ManyToOne
    @JoinColumn(name = "assignedTo", insertable = false, updatable = false)
    private User assignedUser;

    @ManyToOne
    @JoinColumn(name = "requiredSkillId", insertable = false, updatable = false)
    private Skill requiredSkill;
}

// 7. TaskUpdate
public class TaskUpdateDTO { 
    private int taskId; 
    private String comment; 
    private String status; 
}

@Entity
public class TaskUpdate {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int updateId;
    private int taskId;
    private String comment;
    private String status;
    private int updatedBy;
    private LocalDateTime updatedAt;

    @ManyToOne
    @JoinColumn(name = "taskId", insertable = false, updatable = false)
    private Task task;
}

// 8. Report
public class ReportDTO { 
    private int taskId; 
    private String status; 
    private int progressPercent; 
    private String remarks; 
}

@Entity
public class Report {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int reportId;
    private int taskId;
    private String status;
    private int progressPercent;
    private String remarks;
    private Date reportDate;
    private int createdBy;

    @ManyToOne
    @JoinColumn(name = "taskId", insertable = false, updatable = false)
    private Task task;
}

// 9. Notification
public class NotificationDTO { 
    private int userId; 
    private String message; 
    private String type; 
    private boolean isRead; 
}

@Entity
public class Notification {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int notificationId;
    private int userId;
    private String message;
    private String type;
    private boolean isRead;
    private LocalDateTime createdAt;

    @ManyToOne
    @JoinColumn(name = "userId", insertable = false, updatable = false)
    private User user;
}

// 10. SkillHistory
public class SkillHistoryDTO { 
    private int userId; 
    private int skillId; 
    private Date acquiredOn; 
    private String remarks; 
}

@Entity
public class SkillHistory {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int historyId;
    private int userId;
    private int skillId;
    private Date acquiredOn;
    private String remarks;

    @ManyToOne
    @JoinColumn(name = "userId", insertable = false, updatable = false)
    private User user;

    @ManyToOne
    @JoinColumn(name = "skillId", insertable = false, updatable = false)
    private Skill skill;
}

// 11. TeamGroup
public class TeamGroupDTO { 
    private int teamId; 
    private String teamName; 
    private String projectName; 
}

@Entity
public class TeamGroup {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int teamId;
    private String teamName;
    private String projectName;
    private int createdBy;
    private LocalDateTime createdAt;

    @OneToMany(mappedBy = "teamGroup")
    private List<TeamMember> teamMembers;
}

// 12. TeamMember
public class TeamMemberDTO { 
    private int userId; 
    private int teamId; 
    private int skillId; 
    private int taskId; 
    private String roleInTeam; 
}

@Entity
@IdClass(TeamMemberId.class)
public class TeamMember {
    @Id 
    private int userId;
    
    @Id 
    private int teamId;
    
    private int skillId;
    private int taskId;
    private Date joinedOn;
    private String roleInTeam;

    @ManyToOne
    @JoinColumn(name = "userId", insertable = false, updatable = false)
    private User user;

    @ManyToOne
    @JoinColumn(name = "teamId", insertable = false, updatable = false)
    private TeamGroup teamGroup;
}

// 13. TaskRequest
public class TaskRequestDTO { 
    private int userId; 
    private int taskId; 
    private String requestMessage; 
    private String status; 
}

@Entity
public class TaskRequest {
    @Id 
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int requestId;
    private int userId;
    private int taskId;
    private String requestMessage;
    private LocalDateTime requestDate;
    private String status;
    private int respondedBy;
    private LocalDateTime respondedAt;

    @ManyToOne
    @JoinColumn(name = "userId", insertable = false, updatable = false)
    private User user;

    @ManyToOne
    @JoinColumn(name = "taskId", insertable = false, updatable = false)
    private Task task;
}

// 14. UserSkill
public class UserSkillDTO { 
    private int userId; 
    private int skillId; 
    private String proficiencyLevel; 
}

@Entity
@IdClass(UserSkillId.class)
public class UserSkill {
    @Id
    private int userId;

    @Id
    private int skillId;

    private String proficiencyLevel;
    private Date lastUsed;

    @ManyToOne
    @JoinColumn(name = "userId", insertable = false, updatable = false)
    private User user;

    @ManyToOne
    @JoinColumn(name = "skillId", insertable = false, updatable = false)
    private Skill skill;
}

// 15. TaskSkill
public class TaskSkillDTO { 
    private int taskId; 
    private int skillId; 
    private String importanceLevel; 
}

@Entity
@IdClass(TaskSkillId.class)
public class TaskSkill {
    @Id
    private int taskId;

    @Id
    private int skillId;

    private String importanceLevel;

    @ManyToOne
    @JoinColumn(name = "taskId", insertable = false, updatable = false)
    private Task task;

    @ManyToOne
    @JoinColumn(name = "skillId", insertable = false, updatable = false)
    private Skill skill;
}

 

    

---


    

 

    
    
    


    

    
     


