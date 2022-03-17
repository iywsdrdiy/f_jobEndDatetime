USE [Monitor]
GO
SET ANSI_NULLS ON
GO
SET QUOTED_IDENTIFIER ON
GO


create function [dbo].[f_jobEndDatetime]
(@name as nvarchar(128))
--If the scheduler crashed whilst a job was running and the job has not rerun since, the job might even be running in a zombie state and sysjobactivity is unreliable
--If the scheduler has crashed and been restarted while a job was idle and it has not be rerun since, the latest sysjobactivity activity record (for the latest session_id from msdb.dbo.syssessions) will be all null, so you want the one from before that
--Those caveats aside, this is a TSQL function, and so there is no way to raise an error
--Consequently, this function will always return a 'valid' result: if the job doesn't exist you will get a null execution time rather than an error as you might prefer.
returns datetime
as
begin
return(
select stop_execution_date
from msdb.dbo.sysjobs j 
--inner join msdb.dbo.sysjobactivity a on j.job_id = a.job_id and a.session_id = (select session_id from msdb.dbo.syssessions where agent_start_date = (SELECT MAX(agent_start_date) AS max_agent_start_date FROM msdb.dbo.syssessions))
inner join msdb.dbo.sysjobactivity a on j.job_id = a.job_id
inner join (select job_id, max(start_execution_date) latest_start_execution_date from msdb.dbo.sysjobactivity group by job_id) le on a.job_id = le.job_id and a.start_execution_date = le.latest_start_execution_date
left join msdb.dbo.sysjobhistory h on a.job_id = h.job_id and a.job_history_id = instance_id
where j.name = @name)
end
;
GO
