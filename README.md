# mysql_sys.innodb_buffer_stats_by_table-

SELECT 
    IF((LOCATE('.', `ibp`.`TABLE_NAME`) = 0),
        'InnoDB System',
        REPLACE(SUBSTRING_INDEX(`ibp`.`TABLE_NAME`, '.', 1),
            '`',
            '')) AS `object_schema`,
    REPLACE(SUBSTRING_INDEX(`ibp`.`TABLE_NAME`, '.', -(1)),
        '`',
        '') AS `object_name`,
    FORMAT_BYTES(SUM(IF((`ibp`.`COMPRESSED_SIZE` = 0),
                16384,
                `ibp`.`COMPRESSED_SIZE`))) AS `allocated`,
    FORMAT_BYTES(SUM(`ibp`.`DATA_SIZE`)) AS `data`,
    COUNT(`ibp`.`PAGE_NUMBER`) AS `pages`,
    COUNT(IF((`ibp`.`IS_HASHED` = 'YES'), 1, NULL)) AS `pages_hashed`,
    COUNT(IF((`ibp`.`IS_OLD` = 'YES'), 1, NULL)) AS `pages_old`,
    ROUND((SUM(`ibp`.`NUMBER_RECORDS`) / COUNT(DISTINCT `ibp`.`INDEX_NAME`)),
            0) AS `rows_cached`
FROM
    `information_schema`.`INNODB_BUFFER_PAGE` `ibp`
WHERE
    (`ibp`.`TABLE_NAME` IS NOT NULL)
GROUP BY `object_schema` , `object_name`
ORDER BY SUM(IF((`ibp`.`COMPRESSED_SIZE` = 0),
    16384,
    `ibp`.`COMPRESSED_SIZE`)) DESC
