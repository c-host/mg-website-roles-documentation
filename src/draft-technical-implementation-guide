# Technical Implementation Document: Roles-Based Permissioning System for Kirby CMS

## 1. System Overview
This document outlines a draft technical implementation of a roles-based permissioning system for a Kirby CMS-backed website. The system allows for community management of content with an approval workflow.

## 2. Core Components

### 2.1 User Authentication and Roles

```
<?php

Kirby::plugin('yourorganization/custom-auth', [
    'blueprints' => [
        'users/community-member' => __DIR__ . '/blueprints/users/community-member.yml',
        'users/research-contributor' => __DIR__ . '/blueprints/users/research-contributor.yml',
        'users/staff' => __DIR__ . '/blueprints/users/staff.yml',
        'users/admin' => __DIR__ . '/blueprints/users/admin.yml',
    ],
    'hooks' => [
        'user.login:after' => function ($user) {
            $_SESSION['user_roles'] = $user->roles()->toArray();
        }
    ]
]);
```
Create corresponding YAML files for each user role blueprint.

### 2.2 Permission Checks

```
<?php

Kirby::plugin('yourorganization/permission-checks', [
    'hooks' => [
        'page.update:before' => function ($page, $values, $strings) {
            $user = kirby()->user();
            if (!$this->hasPermission($user, $page)) {
                throw new Exception('You do not have permission to edit this page.');
            }
        }
    ],
    'methods' => [
        'hasPermission' => function ($user, $page) {
            $userRoles = $user->roles()->toArray();
            
            if (in_array('admin', $userRoles) || in_array('staff', $userRoles)) {
                return true;
            }
            
            if (in_array('research-contributor', $userRoles)) {
                // Check if the user is listed as a contributor for this project
                return $page->contributors()->toUsers()->has($user);
            }
            
            if (in_array('community-member', $userRoles)) {
                // Community members can only edit their own profile
                return $page->intendedTemplate() == 'profile' && $page->id() == $user->id();
            }
            
            return false;
        }
    ]
]);
```

### 2.3 Content Versioning

```
<?php

Kirby::plugin('yourorganization/content-versioning', [
    'blueprints' => [
        'fields' => [
            'draft_content' => [
                'type' => 'textarea',
                'label' => 'Draft Content'
            ],
            'published_content' => [
                'type' => 'textarea',
                'label' => 'Published Content'
            ]
        ]
    ],
    'hooks' => [
        'page.update:after' => function ($page) {
            // Store the current content as a revision
            $this->storeRevision($page);
        }
    ],
    'methods' => [
        'storeRevision' => function ($page) {
            // Implement revision storage logic
        }
    ]
]);
```

### 2.4 Approval Workflow

```
<?php

Kirby::plugin('yourorganization/approval-workflow', [
    'blueprints' => [
        'fields' => [
            'approval_status' => [
                'type' => 'select',
                'options' => [
                    'pending' => 'Pending',
                    'approved' => 'Approved',
                    'revision_requested' => 'Revision Requested',
                    'rejected' => 'Rejected'
                ],
                'default' => 'pending'
            ],
            'approval_comments' => [
                'type' => 'textarea'
            ]
        ]
    ],
    'hooks' => [
        'page.update:after' => function ($page) {
            if ($page->approval_status()->isNotEmpty() && $page->approval_status()->value() === 'approved') {
                $this->publishContent($page);
            }
        }
    ],
    'routes' => [
        [
            'pattern' => 'api/request-approval/(:any)',
            'action'  => function ($id) {
                return $this->requestApproval($id);
            }
        ],
        [
            'pattern' => 'api/approve/(:any)',
            'action'  => function ($id) {
                return $this->approveContent($id);
            }
        ],
        [
            'pattern' => 'api/request-revision/(:any)',
            'action'  => function ($id) {
                return $this->requestRevision($id);
            }
        ]
    ],
    'methods' => [
        'publishContent' => function ($page) {
            // Implement content publishing logic
        },
        'requestApproval' => function ($id) {
            // Implement approval request logic
        },
        'approveContent' => function ($id) {
            // Implement content approval logic
        },
        'requestRevision' => function ($id) {
            // Implement revision request logic
        }
    ]
]);
```

## 3. Custom Panel Views

### 3.1 Approval Queue
Create a custom Panel view for the approval queue:

```
<?php

return [
    'name' => 'approval-queue',
    'headline' => 'Approval Queue',
    'type' => 'pages',
    'status' => 'pending',
    'templates' => ['article', 'project', 'profile'],
    'sortBy' => 'date desc'
];
```

### 3.2 User Management
Extend the existing users section in the Panel:

```
title: Admin
sections:
  users:
    headline: Users
    type: users
    layout: table
    columns:
      - username
      - email
      - roles
    permissions:
      create: true
      delete: true
```

## 4. Frontend Implementation

### 4.1 Profile Editing
Create a frontend route and template for profile editing:

```
<?php

return [
    [
        'pattern' => 'profile/edit',
        'action'  => function() {
            $user = kirby()->user();
            if (!$user) {
                go('login');
            }
            return page('profile')->render([
                'user' => $user
            ]);
        }
    ]
];
```

### 4.2 Project Page Editing
Implement a custom controller fsor project page editing:

```
<?php

return function ($page) {
    $user = kirby()->user();
    $canEdit = false;

    if ($user) {
        $userRoles = $user->roles()->toArray();
        if (in_array('admin', $userRoles) || in_array('staff', $userRoles)) {
            $canEdit = true;
        } elseif (in_array('research-contributor', $userRoles)) {
            $canEdit = $page->contributors()->toUsers()->has($user);
        }
    }

    return [
        'page' => $page,
        'canEdit' => $canEdit
    ];
};
```

## 5. Notification System

### 5.1 Implement a basic notification system:

```
<?php

Kirby::plugin('yourorganization/notifications', [
    'methods' => [
        'sendNotification' => function ($to, $subject, $message) {
            // Implement notification logic (e.g., email, in-app notification)
        }
    ],
    'hooks' => [
        'page.update:after' => function ($page) {
            if ($page->approval_status()->value() === 'pending') {
                $this->sendNotification('staff@example.com', 'New content pending approval', 'A new page requires your approval.');
            }
        }
    ]
]);
```

## 6. Security Considerations

- Implement proper input sanitization and validation throughout the system.
- Use Kirby's built-in CSRF protection for all forms and API requests.
- Regularly update Kirby CMS and all plugins to the latest versions.
- Implement rate limiting on API routes to prevent abuse.

## 7. Performance Optimization

- Use Kirby's page cache for frequently accessed, non-personalized content.
- Implement database indexing for frequently queried fields (e.g., approval_status).
- Consider implementing a caching layer for the approval queue to reduce database load.

## 8. Testing

- Develop unit tests for critical components (e.g., permission checks, approval workflow).
- Implement integration tests to ensure proper interaction between components.
- Conduct thorough user acceptance testing with representatives from each user role.

## 9. Deployment

- Use version control (e.g., Git) for all custom code and configurations.
- Implement a staging environment that mirrors the production setup for testing.
- Use a deployment script to automate the update process and minimize downtime.

## 10. Maintenance and Monitoring

- Set up logging for critical system events (e.g., failed login attempts, approval actions).
- Implement automated backups of the database and file system.
- Regularly review system logs and performance metrics to identify potential issues.

This technical implementation document provides a comprehensive overview of how to implement the roles-based permissioning system in Kirby CMS. It covers all major components, from user authentication to frontend implementation, and includes considerations for security, performance, testing, deployment, and maintenance.
