# Runtime Verification Commands: Case B

These commands were used to confirm that the CLIENTCHARGE|INACTIVATE
endpoint had never produced a successful response since August 2015.

## Docker image used

docker pull apache/fineract:latest
(verified April 2026)

## HTTP invocation

POST /fineract-provider/api/v1/clients/1/charges/1?command=inactivate

Expected response:
HTTP 400
{"errors":[{"userMessageGlobalisationCode":"error.msg.command.unsupported"}]}

## Git archaeology

Check whether a handler was ever registered:

git log --all -S "clientCharge.inactivate" --oneline

Expected: zero results

Check the origin commit of the stub:

git show d0fd3e4a6c

## SQL verification

Run against the tenant database after the test invocation:

SELECT * FROM m_portfolio_command_source
WHERE action_name = "INACTIVATE" AND entity_name = "CLIENTCHARGE";

Expected: zero rows with made_on_date not null prior to the test run
All rows show status = 5 (rejected before processing)
