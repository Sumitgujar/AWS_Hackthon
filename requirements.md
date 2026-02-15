# Requirements Document: Bharat Setu

## Introduction

Bharat Setu is an AI-powered platform designed for the AI Bharat Hackathon that simplifies access to government schemes, scholarships, internships, and public systems using conversational AI. The platform addresses the critical challenge of scattered, complex information about public opportunities by providing a voice-first (with text fallback) conversational interface that guides users through discovering and understanding government programs.

The system targets students, job seekers, and citizens who struggle to navigate the fragmented landscape of government schemes due to complex language, lack of awareness, and dependency on intermediaries. By leveraging Large Language Models (LLMs) and Speech-to-Text (STT) technology, Bharat Setu democratizes access to public information and opportunities.

## Problem Statement

Citizens, particularly students and job seekers, face significant barriers in accessing government schemes, scholarships, internships, and public systems:

- Information is scattered across multiple government portals and websites
- Complex bureaucratic language makes schemes difficult to understand
- Lack of step-by-step guidance on application processes
- High dependency on intermediaries who may charge fees or provide incomplete information
- Low awareness of available opportunities among target beneficiaries
- Language barriers prevent non-English speakers from accessing information
- Limited internet connectivity in rural and semi-urban areas

## Objectives

1. Simplify complex public information into conversational, easy-to-understand language
2. Provide step-by-step guidance for discovering and applying to schemes
3. Reduce dependency on intermediaries through direct AI-powered assistance
4. Increase awareness of government schemes among students and citizens
5. Make the platform accessible through voice-first interaction with text fallback
6. Support multiple languages (Simple English and Hinglish initially)
7. Enable low-bandwidth access for users with limited connectivity
8. Build a secure, scalable, cloud-ready architecture

## Target Users

### Primary Users
- **Students**: College and university students seeking scholarships, internships, and educational schemes
- **Job Seekers**: Young professionals looking for government job opportunities and skill development programs
- **Rural Citizens**: Individuals in rural and semi-urban areas with limited digital literacy

### Secondary Users
- **Community Leaders**: NGO workers and community organizers helping citizens access schemes
- **Government Officials**: Officials monitoring scheme awareness and adoption

## Glossary

- **System**: The Bharat Setu platform
- **User**: Any individual interacting with the platform (student, job seeker, or citizen)
- **Query**: A question or request submitted by the User through text or voice
- **Scheme**: A government program, scholarship, internship, or public service
- **AI_Assistant**: The conversational AI component powered by LLM
- **STT_Service**: Speech-to-Text service that converts voice input to text
- **Opportunity**: Any scheme, scholarship, internship, or public program available to users
- **Low_Bandwidth_Mode**: A reduced-data mode for users with limited internet connectivity
- **Session**: A continuous interaction period between a User and the System

## Requirements

### Requirement 1: AI Query Assistant

**User Story:** As a user, I want to ask questions about government schemes in natural language, so that I can get clear answers without navigating complex government websites.

#### Acceptance Criteria

1. WHEN a User submits a text query, THE AI_Assistant SHALL process the query and return a relevant response within 5 seconds
2. WHEN a User asks about a specific scheme, THE AI_Assistant SHALL provide information including eligibility criteria, benefits, and application process
3. WHEN a User asks a vague or unclear question, THE AI_Assistant SHALL ask clarifying questions to understand the User's intent
4. WHEN a User requests step-by-step guidance, THE AI_Assistant SHALL provide sequential instructions with clear action items
5. THE AI_Assistant SHALL maintain conversation context across multiple queries within a Session
6. WHEN a User's query cannot be answered with available data, THE AI_Assistant SHALL acknowledge the limitation and suggest alternative resources

### Requirement 2: Voice-Based Query Input

**User Story:** As a user with limited typing ability or digital literacy, I want to speak my questions, so that I can access information without typing.

#### Acceptance Criteria

1. WHEN a User clicks the microphone button, THE System SHALL activate the STT_Service and indicate recording status
2. WHEN a User speaks into the microphone, THE STT_Service SHALL convert speech to text with at least 85% accuracy for clear audio
3. WHEN speech-to-text conversion completes, THE System SHALL display the transcribed text for User confirmation
4. WHEN transcription is incorrect, THE User SHALL be able to edit the text before submission
5. THE System SHALL support voice input in English and Hindi languages
6. WHEN voice input fails or is unavailable, THE System SHALL automatically fall back to text input mode
7. WHEN background noise interferes with recording, THE System SHALL notify the User and suggest retrying in a quieter environment

### Requirement 3: Opportunity Finder

**User Story:** As a student or job seeker, I want to discover relevant schemes and opportunities based on my profile, so that I don't miss out on programs I'm eligible for.

#### Acceptance Criteria

1. WHEN a User provides profile information (age, education, location, category), THE System SHALL filter and display relevant opportunities
2. WHEN displaying opportunities, THE System SHALL show scheme name, eligibility criteria, deadline, and benefits
3. WHEN a User selects an opportunity, THE System SHALL provide detailed information and application guidance
4. THE System SHALL rank opportunities by relevance to the User's profile
5. WHEN new schemes matching a User's profile are added, THE System SHALL notify the User (if notifications are enabled)
6. THE System SHALL support filtering opportunities by category (scholarship, internship, skill development, employment)

### Requirement 4: Public System Explainer

**User Story:** As a citizen unfamiliar with government processes, I want simple explanations of public systems and procedures, so that I can understand how to access government services.

#### Acceptance Criteria

1. WHEN a User asks about a public system or procedure, THE System SHALL provide a simplified explanation in plain language
2. WHEN explaining complex processes, THE System SHALL break down information into digestible steps
3. THE System SHALL provide examples and analogies to clarify abstract concepts
4. WHEN technical or bureaucratic terms are used, THE System SHALL provide definitions in simple language
5. THE System SHALL support queries about common public systems (Aadhaar, PAN, voter registration, ration cards, etc.)

### Requirement 5: Student-Friendly Language

**User Story:** As a student or non-native English speaker, I want information in simple language I can understand, so that I'm not confused by complex terminology.

#### Acceptance Criteria

1. THE AI_Assistant SHALL generate responses using simple English with vocabulary appropriate for 10th-grade reading level
2. THE System SHALL support Hinglish (Hindi-English mix) for users more comfortable with that format
3. WHEN complex terms must be used, THE System SHALL provide explanations or simpler alternatives
4. THE System SHALL avoid bureaucratic jargon and legal language unless specifically requested
5. WHEN a User selects a language preference, THE System SHALL maintain that preference throughout the Session

### Requirement 6: Secure AI Backend

**User Story:** As a platform administrator, I want secure API communication and data handling, so that user information and queries are protected.

#### Acceptance Criteria

1. THE System SHALL encrypt all API communications using TLS 1.3 or higher
2. THE System SHALL validate and sanitize all user inputs before processing
3. THE System SHALL implement rate limiting to prevent abuse (maximum 100 requests per User per hour)
4. WHEN storing user data, THE System SHALL encrypt sensitive information at rest
5. THE System SHALL not log or store personally identifiable information (PII) without explicit user consent
6. THE System SHALL implement authentication for administrative functions
7. WHEN API keys or credentials are used, THE System SHALL store them securely using environment variables or secret management services

### Requirement 7: Cloud-Ready Architecture

**User Story:** As a system architect, I want a scalable cloud-ready architecture, so that the platform can handle varying loads and scale as user base grows.

#### Acceptance Criteria

1. THE System SHALL be deployable on cloud platforms (AWS EC2, Kiro Platform)
2. THE System SHALL use containerization (Docker) for consistent deployment across environments
3. THE System SHALL separate frontend, backend, and AI services into independent components
4. WHEN load increases, THE System SHALL support horizontal scaling of backend services
5. THE System SHALL use stateless API design to enable load balancing
6. THE System SHALL implement health check endpoints for monitoring service availability
7. THE System SHALL log errors and performance metrics for monitoring and debugging

### Requirement 8: Low-Bandwidth Support Mode

**User Story:** As a user in a rural area with limited internet connectivity, I want to access the platform with minimal data usage, so that I can use the service despite poor network conditions.

#### Acceptance Criteria

1. WHEN a User enables Low_Bandwidth_Mode, THE System SHALL reduce data transfer by at least 60%
2. WHILE in Low_Bandwidth_Mode, THE System SHALL disable non-essential images and animations
3. WHILE in Low_Bandwidth_Mode, THE System SHALL compress text responses without losing critical information
4. WHILE in Low_Bandwidth_Mode, THE System SHALL disable voice input and use text-only interaction
5. THE System SHALL automatically detect slow network conditions and suggest enabling Low_Bandwidth_Mode
6. WHEN network conditions improve, THE System SHALL allow Users to switch back to full-feature mode

### Requirement 9: Text Input Fallback

**User Story:** As a user, I want text input to always be available, so that I can use the platform even when voice input is not suitable or available.

#### Acceptance Criteria

1. THE System SHALL provide a visible text input field on all query interfaces
2. WHEN voice input is active, THE System SHALL still allow Users to switch to text input
3. WHEN voice input fails, THE System SHALL automatically enable text input without requiring User action
4. THE System SHALL support text input on all devices and browsers
5. WHEN a User types a query, THE System SHALL process it with the same AI_Assistant as voice queries

### Requirement 10: Response Quality and Accuracy

**User Story:** As a user, I want accurate and up-to-date information about schemes, so that I can make informed decisions and successfully apply to programs.

#### Acceptance Criteria

1. THE AI_Assistant SHALL provide responses based on verified government sources and official documentation
2. WHEN scheme information changes, THE System SHALL update its knowledge base within 48 hours
3. THE System SHALL include source citations or links to official government portals in responses
4. WHEN information accuracy cannot be guaranteed, THE System SHALL include appropriate disclaimers
5. THE System SHALL track response accuracy through user feedback mechanisms
6. WHEN a User reports incorrect information, THE System SHALL flag the response for review

## Non-Functional Requirements

### Performance
- Query response time: < 5 seconds for 95% of requests
- Voice-to-text conversion: < 3 seconds for 30-second audio clips
- Page load time: < 2 seconds on 3G connection
- System uptime: 99.5% availability during hackathon evaluation period

### Scalability
- Support at least 1,000 concurrent users
- Handle 10,000 queries per day
- Scale horizontally to accommodate 10x growth

### Usability
- Mobile-responsive design for screens 320px and above
- Accessible to users with basic digital literacy
- Support for screen readers and accessibility standards (WCAG 2.1 Level AA)
- Maximum 3 clicks to reach any major feature

### Security
- OWASP Top 10 compliance
- Regular security audits and vulnerability scanning
- Secure credential management
- Data encryption in transit and at rest

### Compatibility
- Support modern browsers (Chrome, Firefox, Safari, Edge - latest 2 versions)
- Work on Android 8.0+ and iOS 12+ devices
- Graceful degradation for older browsers

### Maintainability
- Modular architecture with clear separation of concerns
- Comprehensive API documentation
- Code documentation and inline comments
- Version control using Git

## Constraints

### Technical Constraints
- Must use Gemini LLM API for AI capabilities
- Frontend limited to HTML5, CSS3, JavaScript (no frameworks specified)
- Backend must use Python (FastAPI) or Node.js
- Deployment on Kiro Platform or AWS EC2
- Must work within free tier limits of cloud services during development

### Time Constraints
- Development must be completed within hackathon timeline
- MVP must be demo-ready for hackathon presentation

### Resource Constraints
- Limited to hackathon team size and expertise
- Budget constraints for API usage and cloud resources
- No access to proprietary government databases (must use publicly available data)

### Regulatory Constraints
- Must comply with Indian data protection laws
- Cannot store sensitive personal information without consent
- Must include appropriate disclaimers about information accuracy

## Assumptions

1. Users have access to smartphones or computers with internet connectivity
2. Government scheme information is publicly available and can be legally scraped or accessed via APIs
3. Gemini LLM API will remain accessible and within rate limits during development and demo
4. Users are willing to provide basic profile information for personalized recommendations
5. Speech-to-Text services will be available and functional for supported languages
6. The platform will initially focus on central government schemes (state-specific schemes in future scope)
7. Users have basic digital literacy to navigate a web interface

## Future Scope

### Phase 2 Enhancements
- Multi-language support (Tamil, Telugu, Bengali, Marathi, Gujarati, etc.)
- Mobile applications (Android and iOS native apps)
- SMS-based query system for users without smartphones
- Integration with government APIs for real-time scheme updates
- Application tracking system to monitor scheme application status
- Community forum for users to share experiences and tips

### Advanced Features
- AI-powered document verification and application assistance
- Personalized scheme recommendations using machine learning
- Chatbot integration with WhatsApp and Telegram
- Offline mode with cached frequently asked questions
- Video tutorials for complex application processes
- Integration with DigiLocker for document management

### Partnerships
- Collaboration with government departments for official data access
- Partnership with educational institutions for student outreach
- Integration with Common Service Centers (CSCs) for rural access

## Success Metrics

### User Engagement
- Number of unique users during hackathon demo period
- Average session duration > 3 minutes
- Query completion rate > 80%
- User satisfaction rating > 4/5

### Technical Performance
- System uptime > 99.5%
- Average response time < 5 seconds
- Voice recognition accuracy > 85%
- Zero critical security vulnerabilities

### Impact Metrics
- Number of schemes discovered per user > 3
- User feedback indicating improved understanding of schemes > 70%
- Reduction in time to find relevant scheme information (compared to manual search)

### Hackathon Evaluation
- Successful live demo with zero critical failures
- Positive feedback from judges on innovation and social impact
- Demonstration of all core features (voice input, AI query, opportunity finder)
- Clear articulation of scalability and real-world deployment potential
