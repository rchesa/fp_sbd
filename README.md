# FP - SBD

## TRIGGER
```
DELIMITER //
CREATE TRIGGER task_status_update
AFTER UPDATE ON langkah
FOR EACH ROW
BEGIN
    DECLARE langkah_belum_selesai INT;
    SELECT COUNT(*) INTO langkah_belum_selesai
    FROM langkah
    WHERE tugas_id = NEW.tugas_id AND status = 'belum_selesai';

    IF langkah_belum_selesai = 0 THEN
        UPDATE tugas
        SET status = 'selesai'
        WHERE tugas_id = NEW.tugas_id;
    END IF;
END //
DELIMITER ;
```

## VIEW
```
CREATE VIEW langkah_tugas AS
SELECT t.tugas_id, t.nama_tugas, t.jatuh_tempo, t.status, t.kategori, l.langkah, l.status AS status_langkah
FROM tugas t
LEFT JOIN langkah l ON t.tugas_id = l.tugas_id;
```

## STORED PROCEDURE
```
DELIMITER //
CREATE PROCEDURE tambah_tugas(
    IN p_pengguna_id INT,
    IN p_daftar_id INT,
    IN p_nama_tugas VARCHAR(100),
    IN p_jatuh_tempo DATE,
    IN p_kategori ENUM('red', 'yellow', 'green'),
    IN p_file VARCHAR(255)
)
BEGIN
    INSERT INTO tugas (pengguna_id, daftar_id, nama_tugas, jatuh_tempo, status, kategori, file)
    VALUES (p_pengguna_id, p_daftar_id, p_nama_tugas, p_jatuh_tempo, 'belum_selesai', p_kategori, p_file);
END //
DELIMITER ;
```

# FUNCTION
```
DELIMITER //
CREATE FUNCTION tugas_uncomplete(p_pengguna_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE jumlah INT;
    SELECT COUNT(*) INTO jumlah
    FROM tugas
    WHERE pengguna_id = p_pengguna_id AND status = 'belum_selesai';
    RETURN jumlah;
END //
DELIMITER ;
```
