# SANAD - Engineering Instructions

## Product
SANAD is a full-scope medical services
and family-centered care platform.

## Approved Technology Decisions
- Backend: Laravel
- Frontend: One React application
- Database: MySQL (InnoDB)
- Cache/Queue: Redis

## Architecture
- Use Modular Monolith architecture.
- Laravel is the single source of business logic.
- Never introduce a separate Node.js backend
  without explicit approval.
- Maintain strict domain boundaries.
- Follow API-first development.

## Development Rules
- Never implement multiple unrelated features
  in one development task.
- Never invent requirements.
- Refer to source documents before making
  product assumptions.
- Never implement UI with permanent dummy data.
- Never bypass authorization policies.
- Never expose sensitive medical information.
- Never store raw card CVV or PAN.
- Every feature must include automated tests.

## Workflow
1. Inspect existing implementation.
2. Understand the requirement.
3. Propose implementation plan.
4. Identify affected modules and dependencies.
5. Implement the feature.
6. Run tests.
7. Review security and edge cases.
8. Update documentation.
9. Report completed work and remaining issues.

## Git
- Work on dedicated feature branches.
- Never overwrite unrelated changes.
- Never commit secrets.
- Commit only after tests pass.
