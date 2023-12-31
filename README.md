# YeomYuJun.github.io


SELECT BOARD_ID
        , CASE WHEN (DEL_YN = 'Y') THEN '삭제된 글입니다.'
            ELSE TITLE 
            END AS TITLE
        , TITLE AS real
        ,WRITER
        ,BOARD_DEPTH
        ,DEL_YN
        ,fn_final_del(board_id) as f_del_yn
        ,CONNECT_BY_ISLEAF AS LEAF
        ,CONNECT_BY_ROOT BOARD_ID
FROM BOARD 
WHERE 
(DEL_YN = 'N' OR fn_final_del(board_id) ='N') 
START WITH BOARD_DEPTH = 0
CONNECT BY PRIOR BOARD_ID = BOARD_DEPTH
ORDER SIBLINGS BY BOARD_ID DESC;

--------------
--BOARD_DEPTH 찾기 = 포기 깊이 내려가지 못하기 때문에, 자식이 하나라도 있ㅇ면 보이고, 답글들은 삭제될 시 그냥 안보이게 하기로 함
----ROOT 찾기--------
SELECT LISTAGG(DEL_YN, ', ') WITHIN GROUP (order by del_yn) AS FINAL_DEL
FROM BOARD
WHERE BOARD_ID IN(
    SELECT A.BOARD_ID
    FROM 
        (
        SELECT BOARD_ID, CONNECT_BY_ROOT BOARD_ID AS ROOT_ID
        FROM BOARD
        WHERE NOT( CONNECT_BY_ROOT BOARD_ID = BOARD_ID)
        START WITH BOARD_DEPTH = 0
        CONNECT BY PRIOR BOARD_ID = BOARD_DEPTH
        ) A
    WHERE A.ROOT_ID = 15);
    
    
 -------------------------------------

 /*함수
 ==========================================*/   
CREATE OR REPLACE FUNCTION fn_final_del (
    p_board_id NUMBER
)
    RETURN VARCHAR2
IS    
        
      v_del_list VARCHAR2(500);
      v_result board.del_yn%TYPE;
BEGIN
    SELECT LISTAGG(DEL_YN, ', ') WITHIN GROUP (order by del_yn) AS FINAL_DEL
    INTO v_del_list
    FROM BOARD
    WHERE BOARD_ID IN
                (
                SELECT A.BOARD_ID
                FROM 
                    (
                    SELECT BOARD_ID, CONNECT_BY_ROOT BOARD_ID AS ROOT_ID
                    FROM BOARD
                    WHERE NOT( CONNECT_BY_ROOT BOARD_ID = BOARD_ID)
                    START WITH BOARD_DEPTH = 0
                    CONNECT BY PRIOR BOARD_ID = BOARD_DEPTH
                    ) A
                WHERE A.ROOT_ID = p_board_id
                );
                
   IF (INSTR(v_del_list, 'N') > 0) THEN v_del_list := 'N';
   ELSIF v_del_list IS NULL THEN v_del_list := 'O';
   ELSE v_del_list := 'Y';
   END IF;
   RETURN v_del_list;
END;
/
