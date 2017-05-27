# AuditEventRepository
Auditing

   Spring Boot Actuator has a flexible audit framework that will publish events once Spring Security is in play (‘authentication success’, ‘failure’ and ‘access denied’ exceptions by default).
This can be very useful for reporting, and also to implement a lock-out policy based on authentication failures.
To customize published security events you can provide your own implementations of AbstractAuthenticationAuditListener and AbstractAuthorizationAuditListener.

   You can also choose to use the audit services for your own business events.
To do that you can either inject the existing AuditEventRepository into your own components and use that directly,
or you can simply publish AuditApplicationEvent via the Spring ApplicationEventPublisher (using ApplicationEventPublisherAware).