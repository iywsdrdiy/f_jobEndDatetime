# f_jobEndDatetime
Return the datetime the provided job last completed execution

* If the scheduler crashed whilst a job was running and the job has not rerun since, the job might even be running in a zombie state and sysjobactivity is unreliable.
* If the scheduler has crashed and been restarted while a job was _idle_ and it has not be rerun since, the latest sysjobactivity activity record (for the latest session_id from msdb.dbo.syssessions) will be all null, so you want the one from before that.

Those caveats aside, this is a TSQL function and so there is no way to raise an error.  Consequently, this function will always return a valid or safe value: if the job doesn't exist you will get a null execution time rather than an error as you might prefer.

The more you look at the sysjob tables, the more you think designing it was a Friday afternoon job at Microsoft.  The schedule and history model is botchy. In fact, the whole SQL Agent Monitor looks like it was an intern project too.  If you want to know what is truly running now, you can't do it from the sysjob tables because they only at best record what _has_ happened as long as the SQL Agent hasn't crashed.  Most folk seem to trust these or [prefer them](https://am2.co/2016/02/xp_sqlagent_enum_jobs_alt/), but you need the unsupported, undocumented, `xp_sqlagent_enum_jobs` if you want the truth.  I find the SQL Agent crashes once every month or two so this function isn't trustworthy enough (when the SQL Agent crashes and restarts, even the SQL Agent Manager can get running status wrong: we had a job that had been running for months, so that reported, but that also demonstrates we had too many redundant jobs).  For the most accurate picture, look at [fv_jobsForAttention](https://github.com/iywsdrdiy/fv_jobsForAttention).


