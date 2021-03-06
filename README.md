Quality disclaimer
------------------
This is an undocumented and messy PostgreSQL helper thing for PowerShell.
It does not follow any best practices or PowerShell idioms, because it
was written a long time ago. However, it does do the job.

License
-------

See LICENSE.

Installation
------------

You need Npgsql to use this module. See Google for a download location.
For information about how to install modules, see about_Modules.

Usage, sort of
--------------

    & {
        Import-Module Npgsql -DisableNameChecking
        Load-Driver Npgsql.dll
        
        Run-UnitOfWork {
        	# normal queries
            Invoke-Query '
                select customer_id, email_address
                from customers.customer
                where email_address like :email
                  and created < :created' @{
                email = 'niklas@%'
                created = '2011-01-01'
            } | % {
                # do something with $_.customer_id and $_.email_address
                Write-Host "$($_.customer_id) -> $($_.email_address)"
            }
            
            # void queries
            $newCustomerId = Invoke-Query -Void '
                insert into customers.customer
                (email_address, created)
                values (:email, :created)' @{
                email = 'foo@example.com'
                created = [DateTime]::UtcNow
            } -ReturnId 'customers.customer_customer_id_seq'
    
            # scalar queries
            $emailAddress = Invoke-Query -Scalar '
                select email_address
                from customers.customer
                where customer_id = :customerId' @{
                customerId = $newCustomerId
            }
        } -ConnectionString "someconnectionstring" `
          -ClearConnectionPools `
          -Debug `
          #-AutoCommit `
    }

Run-UnitOfWork parameters
-------------------------

ClearConnectionPools clears Npgsql's connection pool before processing begins.

Debug will echo all queries with Write-Host.

AutoCommit will commit when done, unless a problem occurs.
Without -AutoCommit, it will roll back the transaction when done.

Read the horrible source for more information.