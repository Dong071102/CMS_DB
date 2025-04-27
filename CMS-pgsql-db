--
-- PostgreSQL database dump
--

-- Dumped from database version 16.8 (Ubuntu 16.8-0ubuntu0.24.04.1)
-- Dumped by pg_dump version 16.8 (Ubuntu 16.8-0ubuntu0.24.04.1)

-- Started on 2025-04-27 15:37:19 +07

SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
SET client_encoding = 'UTF8';
SET standard_conforming_strings = on;
SELECT pg_catalog.set_config('search_path', '', false);
SET check_function_bodies = false;
SET xmloption = content;
SET client_min_messages = warning;
SET row_security = off;

--
-- TOC entry 2 (class 3079 OID 16388)
-- Name: vector; Type: EXTENSION; Schema: -; Owner: -
--

CREATE EXTENSION IF NOT EXISTS vector WITH SCHEMA public;


--
-- TOC entry 3826 (class 0 OID 0)
-- Dependencies: 2
-- Name: EXTENSION vector; Type: COMMENT; Schema: -; Owner: 
--

COMMENT ON EXTENSION vector IS 'vector data type and ivfflat and hnsw access methods';


--
-- TOC entry 309 (class 1255 OID 16912)
-- Name: check_current_lessons(); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION public.check_current_lessons() RETURNS trigger
    LANGUAGE plpgsql
    AS $$
DECLARE
  max_lessons INTEGER;
BEGIN
  -- ✅ Đúng tên cột: total_lesson (không có "s")
  SELECT total_lesson INTO max_lessons
  FROM courses
  WHERE course_id = NEW.course_id;

  IF NEW.current_lesson > max_lessons THEN
    RAISE EXCEPTION 'Số buổi học vượt quá giới hạn cho phép của khóa học';
  END IF;

  RETURN NEW;
END;
$$;


ALTER FUNCTION public.check_current_lessons() OWNER TO postgres;

--
-- TOC entry 301 (class 1255 OID 17170)
-- Name: get_attendance_pivot(); Type: FUNCTION; Schema: public; Owner: postgres
--

CREATE FUNCTION public.get_attendance_pivot() RETURNS TABLE(student_id uuid, "24-03-2025" text, "20-03-2025" text)
    LANGUAGE plpgsql
    AS $$
DECLARE
    cols TEXT;
    sql  TEXT;
BEGIN
    SELECT string_agg(
             'MAX(CASE WHEN to_char(attendance_time, ''DD-MM-YYYY'') = ''' || att_date || 
             ''' THEN to_char(attendance_time, ''HH24:MI:SS'') END) AS "' || att_date || '"',
             ', '
           )
    INTO cols
    FROM (
        SELECT DISTINCT to_char(attendance_time, 'DD-MM-YYYY') AS att_date
        FROM attendance
    ) sub;

    sql := 'SELECT student_id, ' || cols || ' FROM attendance GROUP BY student_id ORDER BY student_id;';
    
    RAISE NOTICE '%', sql;
    
    RETURN QUERY EXECUTE sql;
END;
$$;


ALTER FUNCTION public.get_attendance_pivot() OWNER TO postgres;

SET default_tablespace = '';

SET default_table_access_method = heap;

--
-- TOC entry 227 (class 1259 OID 17066)
-- Name: admins; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.admins (
    admin_id uuid NOT NULL,
    admin_code character varying(100) NOT NULL,
    face_embedding public.vector(512)
);


ALTER TABLE public.admins OWNER TO postgres;

--
-- TOC entry 225 (class 1259 OID 16846)
-- Name: attendance; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.attendance (
    attendance_id uuid DEFAULT gen_random_uuid() NOT NULL,
    schedule_id uuid NOT NULL,
    student_id uuid NOT NULL,
    status character varying(10) NOT NULL,
    evidence_image_url text,
    attendance_time timestamp without time zone,
    note character varying(255),
    CONSTRAINT attendance_status_check CHECK (((status)::text = ANY ((ARRAY['present'::character varying, 'late'::character varying, 'absent'::character varying])::text[])))
);


ALTER TABLE public.attendance OWNER TO postgres;

--
-- TOC entry 221 (class 1259 OID 16780)
-- Name: cameras; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.cameras (
    camera_id uuid DEFAULT gen_random_uuid() NOT NULL,
    classroom_id uuid NOT NULL,
    camera_url character varying(255) NOT NULL,
    camera_type character varying(20) NOT NULL,
    location character varying(100),
    created_at timestamp without time zone DEFAULT now(),
    socket_path character varying(255) NOT NULL,
    CONSTRAINT cameras_camera_type_check CHECK (((camera_type)::text = ANY ((ARRAY['recognition'::character varying, 'surveillance'::character varying])::text[])))
);


ALTER TABLE public.cameras OWNER TO postgres;

--
-- TOC entry 223 (class 1259 OID 16812)
-- Name: class_students; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.class_students (
    class_id uuid NOT NULL,
    student_id uuid NOT NULL,
    status character varying(255) DEFAULT 'active'::character varying
);


ALTER TABLE public.class_students OWNER TO postgres;

--
-- TOC entry 222 (class 1259 OID 16795)
-- Name: classes; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.classes (
    class_id uuid DEFAULT gen_random_uuid() NOT NULL,
    class_name text,
    course_id uuid,
    lecturer_id uuid,
    created_at timestamp with time zone,
    current_lesson bigint
);


ALTER TABLE public.classes OWNER TO postgres;

--
-- TOC entry 220 (class 1259 OID 16769)
-- Name: classrooms; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.classrooms (
    classroom_id uuid DEFAULT gen_random_uuid() NOT NULL,
    room_name character varying(50) NOT NULL,
    room_type character varying(50) NOT NULL,
    location character varying(100),
    description text,
    created_at timestamp without time zone DEFAULT now()
);


ALTER TABLE public.classrooms OWNER TO postgres;

--
-- TOC entry 219 (class 1259 OID 16757)
-- Name: courses; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.courses (
    course_id uuid DEFAULT gen_random_uuid() NOT NULL,
    course_name text,
    main_lecturer_id uuid NOT NULL,
    created_at timestamp with time zone,
    total_lesson bigint,
    semester_id uuid
);


ALTER TABLE public.courses OWNER TO postgres;

--
-- TOC entry 218 (class 1259 OID 16743)
-- Name: lecturers; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.lecturers (
    lecturer_id uuid NOT NULL,
    lecturer_code character varying(100),
    face_embedding public.vector(512),
    lectainer_code character varying(100)
);


ALTER TABLE public.lecturers OWNER TO postgres;

--
-- TOC entry 226 (class 1259 OID 17007)
-- Name: people_count_snapshots; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.people_count_snapshots (
    snapshot_id uuid DEFAULT gen_random_uuid() NOT NULL,
    schedule_id uuid,
    camera_id uuid NOT NULL,
    people_counter integer DEFAULT 0 NOT NULL,
    captured_at timestamp without time zone DEFAULT CURRENT_TIMESTAMP,
    image_path character varying(255) NOT NULL
);


ALTER TABLE public.people_count_snapshots OWNER TO postgres;

--
-- TOC entry 224 (class 1259 OID 16827)
-- Name: schedules; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.schedules (
    schedule_id uuid DEFAULT gen_random_uuid() NOT NULL,
    class_id uuid NOT NULL,
    classroom_id uuid NOT NULL,
    start_time timestamp without time zone NOT NULL,
    end_time timestamp without time zone NOT NULL,
    topic character varying(255) NOT NULL,
    description text,
    created_at timestamp without time zone DEFAULT now()
);


ALTER TABLE public.schedules OWNER TO postgres;

--
-- TOC entry 228 (class 1259 OID 17173)
-- Name: semesters; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.semesters (
    semester_id uuid NOT NULL,
    semester integer NOT NULL,
    academic_year character varying NOT NULL,
    created_at timestamp with time zone DEFAULT CURRENT_TIMESTAMP
);


ALTER TABLE public.semesters OWNER TO postgres;

--
-- TOC entry 217 (class 1259 OID 16729)
-- Name: students; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.students (
    student_id uuid NOT NULL,
    student_code character varying(100) NOT NULL,
    face_embedding public.vector(512)
);


ALTER TABLE public.students OWNER TO postgres;

--
-- TOC entry 216 (class 1259 OID 16716)
-- Name: users; Type: TABLE; Schema: public; Owner: postgres
--

CREATE TABLE public.users (
    user_id uuid DEFAULT gen_random_uuid() NOT NULL,
    email character varying(255) NOT NULL,
    first_name character varying(100),
    last_name character varying(100) NOT NULL,
    role character varying(50) NOT NULL,
    password_hash character varying(255) NOT NULL,
    created_at timestamp with time zone,
    updated_at timestamp with time zone,
    username character varying(100),
    image_url character varying(255),
    CONSTRAINT users_role_check CHECK (((role)::text = ANY (ARRAY[('student'::character varying)::text, ('lecturer'::character varying)::text, ('admin'::character varying)::text])))
);


ALTER TABLE public.users OWNER TO postgres;

--
-- TOC entry 3819 (class 0 OID 17066)
-- Dependencies: 227
-- Data for Name: admins; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.admins (admin_id, admin_code, face_embedding) FROM stdin;
\.


--
-- TOC entry 3817 (class 0 OID 16846)
-- Dependencies: 225
-- Data for Name: attendance; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.attendance (attendance_id, schedule_id, student_id, status, evidence_image_url, attendance_time, note) FROM stdin;
4cfd114a-f4b9-495e-9773-ecf9843bc498	03302383-cb01-4c8c-bd61-9694809df59b	525fe628-c06f-413e-af2f-b421e5bdcc16	absent	\N	\N	\N
b9f21004-584d-4dc7-8c14-333361a717bc	03302383-cb01-4c8c-bd61-9694809df59b	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
a624975a-4bc5-4cc1-9645-4018a8ab26e3	59135269-924e-41b2-9285-af17c3c9fcfe	04786799-ca23-4231-9cb1-527b8dd7e154	absent	\N	\N	\N
c7d071e0-6d3d-4a02-9633-f66d3df907c4	03302383-cb01-4c8c-bd61-9694809df59b	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
061adfd1-f226-4e63-9ce7-aea00b8e42ae	03302383-cb01-4c8c-bd61-9694809df59b	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
39dc15d0-beb9-4685-b798-549d47e4004c	59135269-924e-41b2-9285-af17c3c9fcfe	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
d40cd44d-87ab-43a4-9210-9512db809fab	03302383-cb01-4c8c-bd61-9694809df59b	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
cbd4886f-a02a-4645-a732-1c2ae5739ff4	59135269-924e-41b2-9285-af17c3c9fcfe	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
a5a75e94-519f-4d20-b346-9da47d73576c	03302383-cb01-4c8c-bd61-9694809df59b	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
3093631d-df0e-47c9-a813-a234ff6a158e	03302383-cb01-4c8c-bd61-9694809df59b	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
1a836abd-aac0-4494-8d86-b4c47de72a27	03302383-cb01-4c8c-bd61-9694809df59b	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
16d9b157-8b7c-4926-9b61-84e69e3d5503	03302383-cb01-4c8c-bd61-9694809df59b	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
5d5bee1f-078b-4092-ac5d-b87b42bb67cc	03302383-cb01-4c8c-bd61-9694809df59b	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
0e2ca6f2-8388-47ac-9951-bff59a976f2a	03302383-cb01-4c8c-bd61-9694809df59b	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
b687bf63-cf0d-4fd1-a3bb-ba3132cee4eb	03302383-cb01-4c8c-bd61-9694809df59b	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
4cab8252-0ad4-40a5-8a2d-73ef0b61b24f	03302383-cb01-4c8c-bd61-9694809df59b	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
bb7d948e-f0fa-47b0-8cd2-df69b12621a7	03302383-cb01-4c8c-bd61-9694809df59b	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
310bcf0e-03aa-46ed-b08c-7080932bef2d	03302383-cb01-4c8c-bd61-9694809df59b	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
700c2f12-c19d-472e-9332-0c5b5b170b0f	03302383-cb01-4c8c-bd61-9694809df59b	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
a2ea96ae-622d-4e44-8394-9d9942ebf041	03302383-cb01-4c8c-bd61-9694809df59b	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
b1afccb6-876c-494f-8a2a-866cbd1f0502	03302383-cb01-4c8c-bd61-9694809df59b	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
f3f6c425-94ee-4cf6-be3c-41eb46fa6d7a	03302383-cb01-4c8c-bd61-9694809df59b	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
2ee09aff-64d5-4dc0-8d94-d8892aabd73f	03302383-cb01-4c8c-bd61-9694809df59b	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
ccf49849-99a4-4946-a07d-939e017a2522	03302383-cb01-4c8c-bd61-9694809df59b	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
0eb55556-30ab-4826-a3d7-f5e2aee88a77	03302383-cb01-4c8c-bd61-9694809df59b	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
19fc1d9f-0de0-4a3a-b690-4c8fc20baf47	03302383-cb01-4c8c-bd61-9694809df59b	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
373de53d-67d2-4fd6-b8b0-52b7ad8439cf	03302383-cb01-4c8c-bd61-9694809df59b	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
fd1dac5a-3297-4c23-abe2-82b9becc9e9a	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	525fe628-c06f-413e-af2f-b421e5bdcc16	absent	\N	\N	\N
3438ddf6-4aab-40b5-b288-ac44c46129c8	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
aea4cee6-fb97-4005-bd47-b347b5d220d4	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
9c01e15e-8d8f-4822-a187-f911f97d7a5d	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
3bba6d80-ccd0-4067-932a-4fa842df8493	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
7fc78d77-6649-44d9-8969-2c1ce536217f	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
bd40558b-ffd2-4815-b9e5-249fdd76c136	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
28f51248-9ffb-4c1b-ad5c-73ae09b284ef	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
22a0562b-0212-4e16-9a3a-e3ba8f678da7	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
9027f75a-1b22-42a3-b3ca-15a9766506a0	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
3a87406e-82d3-4f84-9899-df785860534e	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
94c97b0c-5ed9-4ac9-84e7-22ccbbea611c	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
ec823618-25b2-4ae8-8d5c-e40b6df41bf2	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
45912610-6686-4e5d-a060-6e8c577847f7	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
47d993e5-8e6e-4f13-b77c-fc00e437c083	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
943df9af-04c9-4651-a799-940c899e7f88	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
71605131-dac9-44bd-a9c5-05c5e0c7d8e2	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
b87c8e64-b597-4d83-a55d-60ecc503234b	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
21fed021-d664-48cc-9190-0dfc623e227b	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
2974f42c-823f-4738-9ee8-7daa68785703	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
c46270d7-8101-4893-a072-781e32d5468a	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
673fffe7-bf52-40a3-9fa8-954326f2e7d5	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
6426cc9e-3bec-4b28-bfaf-5392bed75aed	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
18c19344-e158-42f9-9b25-2974314b784a	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
7b2c3482-a1ce-48c0-b5ab-27f7c8b85bd9	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
ec719e2d-eece-4cb4-8a26-04ec2f9df2f1	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
7b6fd739-63b1-4119-a942-3a225e1c2dea	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
aa2b6a0a-f754-4b6a-bbcc-17b4683004f0	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
a56bb640-ea3f-4747-90ca-cd458c70c373	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	04786799-ca23-4231-9cb1-527b8dd7e154	present	evidence_image/2025/03/20/1e9d9b91-b255-4b0a-b7a1-a5e2533eab44_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-03-20 15:16:05	\N
f3deb28c-42ad-4280-bdd3-605eedca0899	03302383-cb01-4c8c-bd61-9694809df59b	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	2025-03-29 16:34:22.268	
702b75f8-08e9-4844-b558-1cc428654e6e	03302383-cb01-4c8c-bd61-9694809df59b	04786799-ca23-4231-9cb1-527b8dd7e154	late	\N	2025-03-29 23:53:08.895	
44d915b2-f90d-4c03-b533-1288a573939d	03302383-cb01-4c8c-bd61-9694809df59b	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	late	\N	2025-03-29 23:54:36.842	
0262f6b0-c5f4-4c12-9f5f-5ee2d123e8fb	03302383-cb01-4c8c-bd61-9694809df59b	f8455952-6f3f-485e-bd9b-942ce5bab472	present	\N	2025-03-29 23:55:16.709	
72925735-329e-4a23-be26-b8128fa435ab	03302383-cb01-4c8c-bd61-9694809df59b	29a519d0-5c29-4496-9687-1b7942dbd7ff	late	\N	2025-03-29 23:56:54.555	
dd926895-8885-48f3-b649-5a5bb77f080f	03302383-cb01-4c8c-bd61-9694809df59b	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	2025-03-29 23:59:32.164	
2580b4d0-e257-47ab-a0b1-7ecb29e3f7bb	1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	f8455952-6f3f-485e-bd9b-942ce5bab472	absent	\N	2025-03-30 00:24:09.77	
4cb1afdb-f1fe-447b-b0ec-45404af27564	fffd3747-6a7d-4a76-8618-72f00704a19e	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
71e14202-338f-4747-9042-21bc6161bf98	fffd3747-6a7d-4a76-8618-72f00704a19e	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
2bd6b87e-8b77-416c-b2bc-7795dc5682b4	fffd3747-6a7d-4a76-8618-72f00704a19e	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
67800cbb-ff2d-4079-bd2f-fe1f20859f15	fffd3747-6a7d-4a76-8618-72f00704a19e	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
4c94fa71-7a2e-4066-b19c-4a53edd096f7	fffd3747-6a7d-4a76-8618-72f00704a19e	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
0a7314c2-dc7e-4e09-91e6-d0d4dce06443	fffd3747-6a7d-4a76-8618-72f00704a19e	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
c79f72b9-ea6d-4f81-94b1-9e8b230b9b97	fffd3747-6a7d-4a76-8618-72f00704a19e	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
83cf3edb-a554-4ed3-bab4-f22976d99b44	fffd3747-6a7d-4a76-8618-72f00704a19e	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
b516d4d3-436c-4304-ad9f-8ab253c1a994	fffd3747-6a7d-4a76-8618-72f00704a19e	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
06a3074e-d173-49fb-ac59-2253498aaea7	fffd3747-6a7d-4a76-8618-72f00704a19e	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
e53327e6-0150-45e0-9d4d-3f9825ee806e	fffd3747-6a7d-4a76-8618-72f00704a19e	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
a995af63-b531-4e7f-b31e-ace1324d62b7	fffd3747-6a7d-4a76-8618-72f00704a19e	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
f51daf3c-7ab6-403a-83a8-1525b91bf58b	fffd3747-6a7d-4a76-8618-72f00704a19e	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
ca5bef53-4ceb-4fa5-ba1c-a3b122c56671	fffd3747-6a7d-4a76-8618-72f00704a19e	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
60be68e9-cfcd-4edc-8a8f-a4b2a3e0f4bd	fffd3747-6a7d-4a76-8618-72f00704a19e	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
8d8bc5f8-a22c-4e37-b997-b75bd60e7c86	fffd3747-6a7d-4a76-8618-72f00704a19e	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
50648a37-86f6-417b-9451-324befa1e39d	fffd3747-6a7d-4a76-8618-72f00704a19e	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
659ababf-50b5-4043-9a75-bbc3d0366f80	fffd3747-6a7d-4a76-8618-72f00704a19e	04786799-ca23-4231-9cb1-527b8dd7e154	present	evidence_image/2025/04/05/fffd3747-6a7d-4a76-8618-72f00704a19e_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-05 09:16:59	\N
00431af2-048e-44e8-9ed2-364fb9c3dbbb	fffd3747-6a7d-4a76-8618-72f00704a19e	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
313f1519-a4ce-4b24-a5fd-39ecc2e5ae8c	fffd3747-6a7d-4a76-8618-72f00704a19e	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
ba93c769-5135-476e-aec9-24772780ac9b	fffd3747-6a7d-4a76-8618-72f00704a19e	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
eb1b6537-d4bc-4e1e-be97-3b10c4f024be	fffd3747-6a7d-4a76-8618-72f00704a19e	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
2a95972d-a841-45b8-a7aa-759b6a9e75b1	fffd3747-6a7d-4a76-8618-72f00704a19e	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
ac1fb1e6-4411-4bf9-99fc-1ce959041a2d	fffd3747-6a7d-4a76-8618-72f00704a19e	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
7419f3f3-5e3c-4c45-82be-8892e396ac13	fffd3747-6a7d-4a76-8618-72f00704a19e	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
3402d98d-60a5-4268-8ae9-388240fe6544	fffd3747-6a7d-4a76-8618-72f00704a19e	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
acb5d4d2-d363-4c6e-8c8d-dbfa0ee0d471	fffd3747-6a7d-4a76-8618-72f00704a19e	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
7b85ffb4-f8e4-48ef-bbcc-70bdc3739774	fffd3747-6a7d-4a76-8618-72f00704a19e	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
cce823f9-7453-4f04-a357-de5feb84a936	59135269-924e-41b2-9285-af17c3c9fcfe	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
ea9ca155-83d2-4b74-b928-9b68aa4aac30	59135269-924e-41b2-9285-af17c3c9fcfe	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
baf0aa9c-1974-43b0-b5f3-3249fa395962	59135269-924e-41b2-9285-af17c3c9fcfe	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
0087ab5d-1426-476b-9c51-e0ab2ca1064d	59135269-924e-41b2-9285-af17c3c9fcfe	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
f822be7c-3d24-448e-bb3d-a02fc9a7a2d9	59135269-924e-41b2-9285-af17c3c9fcfe	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
a232834a-9ede-477b-8316-c3750153db56	59135269-924e-41b2-9285-af17c3c9fcfe	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
e8a7e2b9-b8db-40d1-9906-9e98defc00fc	59135269-924e-41b2-9285-af17c3c9fcfe	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
4e84daae-f1f5-4d37-90e8-16903de46bcc	59135269-924e-41b2-9285-af17c3c9fcfe	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
3fd322b1-af3a-416b-945b-e5b9e35a35e0	59135269-924e-41b2-9285-af17c3c9fcfe	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
ae93a594-553b-4581-b20c-a3b1fdc85f52	59135269-924e-41b2-9285-af17c3c9fcfe	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
12269316-2d1c-4722-91d8-4ae33800f9a0	59135269-924e-41b2-9285-af17c3c9fcfe	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
48725031-da8f-4644-9bc3-7d11881fb0c3	59135269-924e-41b2-9285-af17c3c9fcfe	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
0e7c7f1f-8d01-4abb-8ea5-769f27ea7a3c	59135269-924e-41b2-9285-af17c3c9fcfe	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
99430520-63be-446c-9495-ce272d19d801	59135269-924e-41b2-9285-af17c3c9fcfe	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
419d66f7-cec6-4f90-a142-1c96fa90b141	59135269-924e-41b2-9285-af17c3c9fcfe	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
70f66b4c-fc05-4568-8598-7d0c959c224d	59135269-924e-41b2-9285-af17c3c9fcfe	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
26d75801-8d4f-4dda-a635-1fb44c2092ae	59135269-924e-41b2-9285-af17c3c9fcfe	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
3695a1a0-0448-4857-bddb-673296e48eff	59135269-924e-41b2-9285-af17c3c9fcfe	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
f9589a8e-f94b-4e0a-9357-8c6e230d0075	59135269-924e-41b2-9285-af17c3c9fcfe	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
c7d3dbfc-aa92-4a6f-8252-6ef172156591	59135269-924e-41b2-9285-af17c3c9fcfe	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
5f589534-17a0-49c4-8132-73744641bf5a	59135269-924e-41b2-9285-af17c3c9fcfe	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
fd0339b3-6506-4675-a964-207e1de8bd03	59135269-924e-41b2-9285-af17c3c9fcfe	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
3607b14c-d736-48d2-8cb0-13af686befbc	59135269-924e-41b2-9285-af17c3c9fcfe	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
2f1e4a31-f696-4d1e-9798-fd42b89c9455	59135269-924e-41b2-9285-af17c3c9fcfe	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
97b13e4a-ee9b-4ee2-b795-4bb64fdd1cd7	59135269-924e-41b2-9285-af17c3c9fcfe	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	present	\N	2025-04-06 15:52:41.468	
6720c956-82c4-4c42-a969-b97ecb5fb00d	02516f2d-f313-4464-aa1d-01704c6d6ad3	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
30e16c6f-1085-4381-a7ec-1ab74da3f5d5	02516f2d-f313-4464-aa1d-01704c6d6ad3	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
5d70eb78-2a05-43f0-baf3-66cbd47f6294	02516f2d-f313-4464-aa1d-01704c6d6ad3	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
ac5e7c3a-7f99-4bbb-8eaf-f3303d89e371	02516f2d-f313-4464-aa1d-01704c6d6ad3	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
c370ea0a-f2cd-4d82-bec3-9c3144143955	02516f2d-f313-4464-aa1d-01704c6d6ad3	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
1aa98c57-76cb-4e79-8889-6e8f3cd1f5a0	02516f2d-f313-4464-aa1d-01704c6d6ad3	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
d6f87569-051a-40fc-8d28-9ab512ded501	02516f2d-f313-4464-aa1d-01704c6d6ad3	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
cd0cd18b-c3b6-40e8-bff2-5ade695eaa53	02516f2d-f313-4464-aa1d-01704c6d6ad3	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
369cc0dc-cc54-49e2-b15d-02fedf5359d1	02516f2d-f313-4464-aa1d-01704c6d6ad3	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
825d2a33-0ec3-44de-81e6-5d9a8ad68624	02516f2d-f313-4464-aa1d-01704c6d6ad3	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
42e52cf3-22b1-42a9-ab35-17f8e8c68b7f	02516f2d-f313-4464-aa1d-01704c6d6ad3	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
ca965b34-42ad-4320-b4d2-fb31ddafedfe	02516f2d-f313-4464-aa1d-01704c6d6ad3	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
6d0af879-377e-430b-9e7a-1f770881f51b	02516f2d-f313-4464-aa1d-01704c6d6ad3	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
871c1e20-4580-4c06-9978-71ff90aa1e48	02516f2d-f313-4464-aa1d-01704c6d6ad3	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
f1790b51-088c-492f-a4d4-686e575f6f40	02516f2d-f313-4464-aa1d-01704c6d6ad3	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
fe7e36d4-0003-4a4d-a1f6-3e5777b81b91	02516f2d-f313-4464-aa1d-01704c6d6ad3	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
b0cb3c83-2384-429c-853e-b26cc6b7eca3	02516f2d-f313-4464-aa1d-01704c6d6ad3	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
bd6b132a-2a4f-4468-8925-ce15b765606f	02516f2d-f313-4464-aa1d-01704c6d6ad3	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
41ed3244-6aa8-464d-8965-5a8ef6df0901	02516f2d-f313-4464-aa1d-01704c6d6ad3	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
ecd6bc30-814b-46da-94ae-d6749a38d86e	02516f2d-f313-4464-aa1d-01704c6d6ad3	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
17633cef-634e-4890-8cb6-47d8aaecbfc6	02516f2d-f313-4464-aa1d-01704c6d6ad3	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
25fa6f70-bf0c-4456-9cb2-77fb7a4595b8	02516f2d-f313-4464-aa1d-01704c6d6ad3	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
06c73f53-6555-4abd-976d-b6db4a5b8f3a	02516f2d-f313-4464-aa1d-01704c6d6ad3	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
65b7bfb0-b3bf-4a1c-937b-7a14226aa34f	02516f2d-f313-4464-aa1d-01704c6d6ad3	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
937b953f-7cab-49d5-9266-77ce65da9a3a	02516f2d-f313-4464-aa1d-01704c6d6ad3	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
7e48cb11-2b71-416d-9183-6cc7794da275	02516f2d-f313-4464-aa1d-01704c6d6ad3	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
b57ae79f-faff-4163-abff-e82a003d2a38	02516f2d-f313-4464-aa1d-01704c6d6ad3	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
043d0fbd-31a9-4025-a33a-76c5e1c7b31b	02516f2d-f313-4464-aa1d-01704c6d6ad3	04786799-ca23-4231-9cb1-527b8dd7e154	present	evidence_image/2025/04/09/02516f2d-f313-4464-aa1d-01704c6d6ad3_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-09 15:19:42	\N
912f4864-8e3c-46a9-b5b7-dc5a096651cb	587da79e-2b3c-46be-9c4c-42e218baff27	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
b3049165-a1fd-4d64-8818-f90a6c39ddc7	587da79e-2b3c-46be-9c4c-42e218baff27	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
5dce03a3-0952-45f5-851f-6255424b31ce	587da79e-2b3c-46be-9c4c-42e218baff27	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
3bc6ad2d-32d0-4bd1-adef-3946a1f40d06	587da79e-2b3c-46be-9c4c-42e218baff27	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
2436d032-f2c8-4960-bdb0-8b5d451210b5	587da79e-2b3c-46be-9c4c-42e218baff27	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
e9bdbbc2-83bb-4bfe-8a2c-0fc12b6a7260	587da79e-2b3c-46be-9c4c-42e218baff27	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
c26306ba-3d38-4c32-a7ee-60e97510d106	587da79e-2b3c-46be-9c4c-42e218baff27	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
37b4b3d4-1b1a-47f0-9488-d59cdcacb679	587da79e-2b3c-46be-9c4c-42e218baff27	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
efaf8a8e-1d10-42aa-ad03-33ae514678bf	587da79e-2b3c-46be-9c4c-42e218baff27	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
6d5edacf-9af4-451e-b6f7-2d9b28c432b9	587da79e-2b3c-46be-9c4c-42e218baff27	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
2842cbd9-1767-404e-86ae-867b47eaed84	587da79e-2b3c-46be-9c4c-42e218baff27	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
76e641bc-2de0-4e88-bbef-b6b1ad7c1069	587da79e-2b3c-46be-9c4c-42e218baff27	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
66ac21f9-407f-41ae-b4e1-0785105ce87d	587da79e-2b3c-46be-9c4c-42e218baff27	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
0835ae7e-842e-4742-bf3e-a8857dd3cdc3	587da79e-2b3c-46be-9c4c-42e218baff27	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
c1bd64c1-46f2-4581-9f86-d932fc024d72	587da79e-2b3c-46be-9c4c-42e218baff27	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
4acb4d75-726e-4267-a030-cd6a0ab42583	587da79e-2b3c-46be-9c4c-42e218baff27	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
2e85ae26-4375-4d33-b3b3-ee5e2c7c2045	587da79e-2b3c-46be-9c4c-42e218baff27	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
b420e5bd-013a-443e-b2c2-f21b237bf4d9	587da79e-2b3c-46be-9c4c-42e218baff27	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
edb7e6ef-21b2-4cdd-896f-7f9ca9d308d2	587da79e-2b3c-46be-9c4c-42e218baff27	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
6fb761b7-1680-4506-b928-1e6c0f3dc22a	587da79e-2b3c-46be-9c4c-42e218baff27	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
d57529c6-da96-4505-8fb5-374e8020cc25	587da79e-2b3c-46be-9c4c-42e218baff27	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
45ff7320-d2c0-4604-b749-d19e3b2bdccd	587da79e-2b3c-46be-9c4c-42e218baff27	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
b2e3bdf4-4ab4-40b1-a154-ff79a95e9d99	587da79e-2b3c-46be-9c4c-42e218baff27	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
b7253fa9-061f-4a5f-9b9d-0eb8653a84a3	587da79e-2b3c-46be-9c4c-42e218baff27	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
d39346f3-4136-4d69-bc5c-b94dd60b97f2	587da79e-2b3c-46be-9c4c-42e218baff27	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
f363538f-4d92-438f-8fac-f47c9b131f54	587da79e-2b3c-46be-9c4c-42e218baff27	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
81ff79a2-7c46-4015-b951-2cc23c5cfb52	587da79e-2b3c-46be-9c4c-42e218baff27	04786799-ca23-4231-9cb1-527b8dd7e154	present	evidence_image/2025/04/10/587da79e-2b3c-46be-9c4c-42e218baff27_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-10 13:49:59	\N
aa14f2ee-d524-49e8-a899-92211d4082ff	587da79e-2b3c-46be-9c4c-42e218baff27	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	present	\N	2025-04-13 13:01:48.922	
39b3a789-7b2d-434b-bdf7-2a6ee063fb34	1f7a6921-9fdd-4087-82b5-66aa953ae42d	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
48072e78-d04a-4ab5-9b11-2382b9cfc0f2	1f7a6921-9fdd-4087-82b5-66aa953ae42d	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
b02156cf-4291-4444-9ba0-f831694269d0	1f7a6921-9fdd-4087-82b5-66aa953ae42d	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
a5d22e6e-fd05-4297-aa17-b272a29fb1c0	1f7a6921-9fdd-4087-82b5-66aa953ae42d	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
7b5c352d-da99-4cab-ab63-5593f793c031	1f7a6921-9fdd-4087-82b5-66aa953ae42d	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
690bdfb0-1de7-4923-a614-aa9e312c057e	1f7a6921-9fdd-4087-82b5-66aa953ae42d	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
67fae901-343b-497e-80ae-eeeeabf7937c	1f7a6921-9fdd-4087-82b5-66aa953ae42d	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
99c6a0bf-15e0-46dc-8fed-58b60f3f774c	1f7a6921-9fdd-4087-82b5-66aa953ae42d	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
eac390c4-78f5-4cb4-8bc6-fed8a160f14b	1f7a6921-9fdd-4087-82b5-66aa953ae42d	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
a33331f1-c1ec-4b5b-8c4d-f0bbd32e3970	1f7a6921-9fdd-4087-82b5-66aa953ae42d	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
ff752d30-eea3-4abd-9cdf-348e504b1ae8	1f7a6921-9fdd-4087-82b5-66aa953ae42d	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
069ed0ca-65d4-4d91-b533-2bb921e89922	1f7a6921-9fdd-4087-82b5-66aa953ae42d	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
77ba8fb1-61b4-44bc-9644-dc8bb3ad3c78	1f7a6921-9fdd-4087-82b5-66aa953ae42d	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
e3ceb179-87d9-443f-90f5-0fa3a4162247	1f7a6921-9fdd-4087-82b5-66aa953ae42d	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
e43bf82b-c6a3-43c9-9393-3c07e8c4c28d	1f7a6921-9fdd-4087-82b5-66aa953ae42d	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
7f227e52-ab35-478d-89cd-1426cedd0042	1f7a6921-9fdd-4087-82b5-66aa953ae42d	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
ae0f6121-5034-46da-ac14-60c412947081	1f7a6921-9fdd-4087-82b5-66aa953ae42d	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
b6496376-17a1-4ce7-be08-9154dbd8e7a0	1f7a6921-9fdd-4087-82b5-66aa953ae42d	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
7b299951-646a-4d3c-8b4d-5b0dff8d5a29	1f7a6921-9fdd-4087-82b5-66aa953ae42d	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
bca41f9b-d417-4afa-9066-54d4876e762d	1f7a6921-9fdd-4087-82b5-66aa953ae42d	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
c4b1051a-b32f-4629-9925-f19ec70882f9	1f7a6921-9fdd-4087-82b5-66aa953ae42d	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
5aa61e66-ab79-47fe-900c-bbda963d8151	1f7a6921-9fdd-4087-82b5-66aa953ae42d	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
c9fe953f-f7a9-4033-afad-ef53b71cb828	1f7a6921-9fdd-4087-82b5-66aa953ae42d	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
49cf9a5e-2ad2-4dfe-8c4e-b00d6b62597e	1f7a6921-9fdd-4087-82b5-66aa953ae42d	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
e154b962-f8c2-4847-84f9-1963dd5c40ab	1f7a6921-9fdd-4087-82b5-66aa953ae42d	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
21c26187-d36f-4dd3-9f13-0a2a955fdd6b	1f7a6921-9fdd-4087-82b5-66aa953ae42d	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
efc3a4af-4315-42ec-8cf5-42e5a4be06ae	1f7a6921-9fdd-4087-82b5-66aa953ae42d	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
d34f9af7-37c1-4ea2-aa86-b74099e74451	1f7a6921-9fdd-4087-82b5-66aa953ae42d	04786799-ca23-4231-9cb1-527b8dd7e154	present	src/evidence_image/2025/04/13/1f7a6921-9fdd-4087-82b5-66aa953ae42d_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-13 20:22:43	\N
0578d62c-0e6e-416d-8e1c-52f9beb3ea33	279be7ea-a84a-49f9-8b33-43419e3eb271	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
e66046ec-baf6-4515-8a8f-31a9ae9a7e9c	279be7ea-a84a-49f9-8b33-43419e3eb271	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
b546e6c9-5d79-4051-8929-d672af668058	279be7ea-a84a-49f9-8b33-43419e3eb271	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
b9b3a2ff-3e32-49c0-868e-93111bdb7939	279be7ea-a84a-49f9-8b33-43419e3eb271	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
3186f088-88e6-4bad-8356-ae590f1cdede	279be7ea-a84a-49f9-8b33-43419e3eb271	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
3e871b02-1010-48f2-b78c-1ec080b5ca2e	279be7ea-a84a-49f9-8b33-43419e3eb271	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
38144222-052c-4ac6-8f53-6639a906a85a	279be7ea-a84a-49f9-8b33-43419e3eb271	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
4e33b50b-b620-44b9-a4cb-e91604e495e1	279be7ea-a84a-49f9-8b33-43419e3eb271	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
e58c82ab-9af5-4d92-91f4-c62fdbe85b48	279be7ea-a84a-49f9-8b33-43419e3eb271	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
86bdcaef-fe95-4490-a50d-404bfe83ddfe	279be7ea-a84a-49f9-8b33-43419e3eb271	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
821501ca-2a97-4001-9906-d9aa693bfeaa	279be7ea-a84a-49f9-8b33-43419e3eb271	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
866a0e2b-0b66-4db4-95d0-d4155577f335	279be7ea-a84a-49f9-8b33-43419e3eb271	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
9dec80ed-7652-4e6e-94fe-5ecd22044e49	279be7ea-a84a-49f9-8b33-43419e3eb271	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
e9f915f6-218e-4d49-9df6-dd587e74e1dd	279be7ea-a84a-49f9-8b33-43419e3eb271	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
a8bf2284-fbb7-4cb1-b2f9-9c825d256a13	279be7ea-a84a-49f9-8b33-43419e3eb271	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
447263fa-b488-477d-80ae-a080b6175975	279be7ea-a84a-49f9-8b33-43419e3eb271	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
0c540a19-782d-429e-86a1-ea04858c9493	279be7ea-a84a-49f9-8b33-43419e3eb271	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
a4d843a2-1296-4728-8360-6721773016df	279be7ea-a84a-49f9-8b33-43419e3eb271	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
32d54b76-3cf3-4701-8ddb-d398217a7bb5	279be7ea-a84a-49f9-8b33-43419e3eb271	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
9ad48277-f0c8-4191-94e2-96fffe63544f	279be7ea-a84a-49f9-8b33-43419e3eb271	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
b5d3d579-2973-4cae-9c76-7feba7e16e83	279be7ea-a84a-49f9-8b33-43419e3eb271	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
8eb45a21-7c68-4101-913b-5ee84224e85d	279be7ea-a84a-49f9-8b33-43419e3eb271	04786799-ca23-4231-9cb1-527b8dd7e154	present	src/evidence_image/2025/04/16/279be7ea-a84a-49f9-8b33-43419e3eb271_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-16 20:31:32	\N
7bb6073f-6cdc-4d67-9e89-4d6af71e0acb	279be7ea-a84a-49f9-8b33-43419e3eb271	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
ed452702-597e-4b79-a926-814a3661fdcf	279be7ea-a84a-49f9-8b33-43419e3eb271	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
f68774e8-3056-4da6-877a-cfbe066e0c50	279be7ea-a84a-49f9-8b33-43419e3eb271	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
657c601c-1e1c-48b9-8ab9-267d777a0915	279be7ea-a84a-49f9-8b33-43419e3eb271	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
bee287cf-92af-4608-b640-0d2ce3941283	279be7ea-a84a-49f9-8b33-43419e3eb271	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
fe70a7a9-f53d-466d-b9c3-29d51d9e5b90	279be7ea-a84a-49f9-8b33-43419e3eb271	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
673106ff-b3f9-471b-8bbe-23a58bd5a49e	951c502d-abe6-4c5b-9b5c-8f563a0b551f	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
ff0bba87-115e-4806-8c57-99083d824654	951c502d-abe6-4c5b-9b5c-8f563a0b551f	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
ef5ab6ef-db1b-4554-ac92-adec89caf61d	951c502d-abe6-4c5b-9b5c-8f563a0b551f	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
02fee6bc-3aa0-4a20-911f-d9cc3be5cc97	951c502d-abe6-4c5b-9b5c-8f563a0b551f	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
2359831c-e50f-4be6-bf2d-631a3d38bd9b	951c502d-abe6-4c5b-9b5c-8f563a0b551f	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
74fa21d2-e1a1-4c87-86d4-0ba04d6d9025	951c502d-abe6-4c5b-9b5c-8f563a0b551f	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
bcd2bb38-821d-491c-bf5e-a572c57189ec	951c502d-abe6-4c5b-9b5c-8f563a0b551f	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
f17ff009-8c30-483d-b4fb-969bf6852e6f	951c502d-abe6-4c5b-9b5c-8f563a0b551f	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
9ff854e2-6347-4c5d-b8d6-02a733423f83	951c502d-abe6-4c5b-9b5c-8f563a0b551f	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
e8ca0d2f-52d5-4db2-b273-243b503d1b9d	951c502d-abe6-4c5b-9b5c-8f563a0b551f	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
9beb62b5-2274-49b0-85b4-56ebb6e62bed	951c502d-abe6-4c5b-9b5c-8f563a0b551f	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
5595c102-7e50-4098-b655-01fc20a241a2	951c502d-abe6-4c5b-9b5c-8f563a0b551f	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
c521fb53-d24d-4538-8acf-9dff5116b83e	951c502d-abe6-4c5b-9b5c-8f563a0b551f	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
f274e9a6-fc92-41a8-9dcf-d65584f54321	951c502d-abe6-4c5b-9b5c-8f563a0b551f	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
46bf7211-beb6-4ae0-847c-5d7f8d9e99d7	951c502d-abe6-4c5b-9b5c-8f563a0b551f	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
9a4beb57-6fcb-4d97-93f2-ee37f2e8ccf5	951c502d-abe6-4c5b-9b5c-8f563a0b551f	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
6bf6802f-a005-44a6-8980-8d57c754a9d8	951c502d-abe6-4c5b-9b5c-8f563a0b551f	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
f45c5ec0-6208-46d3-9cb1-b70aba24bf44	951c502d-abe6-4c5b-9b5c-8f563a0b551f	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
d85d07ce-d5e2-4b24-9cc9-87d15d1b8102	951c502d-abe6-4c5b-9b5c-8f563a0b551f	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
a23b0b77-352a-4278-bdae-3e30ba6b59e4	951c502d-abe6-4c5b-9b5c-8f563a0b551f	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
705a7783-ce95-4ef4-8cd2-e9c31a353d30	951c502d-abe6-4c5b-9b5c-8f563a0b551f	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
bd2e824e-c25b-4ce3-a403-3deb1b34e76d	951c502d-abe6-4c5b-9b5c-8f563a0b551f	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
4b85a48b-bf53-4797-b486-4ff092d9957b	951c502d-abe6-4c5b-9b5c-8f563a0b551f	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
a1380eea-b0b0-4987-bc82-d47d1ac24c0f	951c502d-abe6-4c5b-9b5c-8f563a0b551f	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
38399f07-ffaa-4c3a-af17-3672895809b2	951c502d-abe6-4c5b-9b5c-8f563a0b551f	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
183c841f-0c01-47a2-9b83-502263b6d893	951c502d-abe6-4c5b-9b5c-8f563a0b551f	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
ffbbf779-a2c8-45c4-be7f-ca74a7de83a5	951c502d-abe6-4c5b-9b5c-8f563a0b551f	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
10554fd6-167f-406b-bae6-b324f0ca5ee1	951c502d-abe6-4c5b-9b5c-8f563a0b551f	04786799-ca23-4231-9cb1-527b8dd7e154	late	\N	2025-04-17 06:50:28.672	
6643797a-739b-4a71-9ec5-cdd9e6230728	c81c61d9-8b39-4dcf-93f4-1b172267506e	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
fcf08842-1adb-4aec-b42e-e6fd19861eb6	c81c61d9-8b39-4dcf-93f4-1b172267506e	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
dd580906-66f3-4729-ac52-d6643adc7277	c81c61d9-8b39-4dcf-93f4-1b172267506e	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
32d46e1c-c9d7-4d3b-bc52-c75a5fc3197b	c81c61d9-8b39-4dcf-93f4-1b172267506e	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
65ef910a-b10b-44c3-b558-2bdf876d483d	c81c61d9-8b39-4dcf-93f4-1b172267506e	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
19967b22-b4b8-45bb-b215-f0c892c4ebdf	c81c61d9-8b39-4dcf-93f4-1b172267506e	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
39966563-7e8f-4811-8801-43620afe23a4	c81c61d9-8b39-4dcf-93f4-1b172267506e	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
4ecbc327-58b5-4caa-823e-3c050e0e386e	c81c61d9-8b39-4dcf-93f4-1b172267506e	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
551c7816-298a-464c-a4f9-7dac7652684d	c81c61d9-8b39-4dcf-93f4-1b172267506e	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
5212b10a-ce98-4b00-93d7-8151f0fd2b89	c81c61d9-8b39-4dcf-93f4-1b172267506e	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
756fe0e1-0500-4d54-9711-7a69bfaada4b	c81c61d9-8b39-4dcf-93f4-1b172267506e	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
49ca14b8-f5ab-4743-895f-71703e6a6d2a	c81c61d9-8b39-4dcf-93f4-1b172267506e	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
f291536c-1e1b-422a-84cb-a0857faca156	c81c61d9-8b39-4dcf-93f4-1b172267506e	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
6f7322b5-72b6-48eb-bea1-5e4087c7d384	c81c61d9-8b39-4dcf-93f4-1b172267506e	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
0b9c53b0-aca5-4164-b831-dd72fcec6be4	c81c61d9-8b39-4dcf-93f4-1b172267506e	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
ceaf6918-4156-4e53-9210-69ed9f4640c9	c81c61d9-8b39-4dcf-93f4-1b172267506e	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
be38f800-5787-47b9-9c99-892d63f1bdb4	c81c61d9-8b39-4dcf-93f4-1b172267506e	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
e8b5317e-a03c-49cb-a9bf-76078e768cf1	c81c61d9-8b39-4dcf-93f4-1b172267506e	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
7a069bd8-725e-43ef-98b0-7f674cf09fbe	c81c61d9-8b39-4dcf-93f4-1b172267506e	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
db9e0d8a-e110-44c3-bffd-6ee9e9c4316b	c81c61d9-8b39-4dcf-93f4-1b172267506e	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
ebdd8585-d119-474f-94ed-a3d0f98e765b	c81c61d9-8b39-4dcf-93f4-1b172267506e	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
113ffc2b-29b3-49ec-9e22-df3a92632124	c81c61d9-8b39-4dcf-93f4-1b172267506e	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
efeb28e3-0142-4757-b971-7cb7999b9a49	c81c61d9-8b39-4dcf-93f4-1b172267506e	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
5e33a033-dc56-4f14-b04f-0cc3a98b46ad	c81c61d9-8b39-4dcf-93f4-1b172267506e	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
76a9147b-d944-4fda-9e24-eb70be263b12	c81c61d9-8b39-4dcf-93f4-1b172267506e	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
7037f46d-fc46-4b33-9d89-5687e11cedfc	c81c61d9-8b39-4dcf-93f4-1b172267506e	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
cab8753a-b558-4f7a-a188-70bc12d0f1be	c81c61d9-8b39-4dcf-93f4-1b172267506e	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
e69c2c93-f48b-46a8-a770-115ff045588c	c81c61d9-8b39-4dcf-93f4-1b172267506e	04786799-ca23-4231-9cb1-527b8dd7e154	present	src/evidence_image/2025/04/23/c81c61d9-8b39-4dcf-93f4-1b172267506e_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-23 11:16:36	\N
b0b4890f-20e9-4377-b1b0-88a748757843	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
d7eb151e-3f0b-440b-a781-354d998211e2	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
18ffcba6-0970-424d-b59f-1be7ced8d161	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
06584db2-3f96-4090-b9af-01c4c163961f	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
38178e30-8f96-45da-b5e3-d81830f7b7a2	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
1570a4cc-8842-42d6-8905-921cff0680fa	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
bb78f4f5-3c11-4c89-9d16-1ecf4e030473	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
f4c25aef-0908-408a-8837-668e9fc6ed80	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
acd07b74-c6fa-48a0-8db2-47968176a079	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
9a0f7302-60a7-4565-ba24-cb88c02b0a4d	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
fd88ca9d-7ff2-4f3e-8797-722ef16554de	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
eae63175-8770-484d-b3da-f302e5258041	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
ec7e9a97-450e-483a-a498-ec3a9fa6ba45	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
feac2683-10da-4f87-8e91-a49bea2af0f3	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
af0a8642-331f-4554-aa08-2ab70d7dbe9b	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
ea19df39-ce78-4a3d-856d-6906387f3fdf	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
f2a72073-c45f-49d0-bc0a-6142591cb51e	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
a6b8aebb-ad86-4a2e-8220-05b006caa159	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	04786799-ca23-4231-9cb1-527b8dd7e154	present	src/evidence_image/2025/04/23/c92ab6b7-ba43-4d02-9b1e-cc462cbeb672_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-23 20:04:12	\N
91ce33af-271d-46bf-967b-fe58461f6ed8	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
8a436b95-59a0-45b0-a33c-d17ec4f008ba	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
098055f7-df3e-4884-8b14-e39d3a0cc150	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
e10f42fa-0f0b-4b88-8744-cdcf09378b4d	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
4175f4be-0796-4e5c-b6ab-ac969053694f	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
1c02cb48-72a8-46d1-967e-3b942115d2c1	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
ce11b1b2-ba8a-4089-9ba3-26c836062ef0	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
c0f56ef0-f569-42c8-99a0-c909f92af83e	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
1a5cfd38-3064-414e-ab17-6db386c84058	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
3bd5bbf8-3497-4154-8895-81a8858c7553	c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
5bc0c109-4266-4d88-bde6-9ad2429bd5e5	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	absent	\N	\N	\N
1eaf5751-20c2-40c7-8bb0-9789b1f1614d	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
67833716-88b9-471b-a4a8-e71c19666b77	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
db96a532-2c3a-485f-8f3b-5909eac0945a	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
d3ed5f24-dbb8-4124-a28e-7dc9f9eb927d	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
eca1f8c9-8887-4b84-938a-88770acb53f8	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
fedeee52-cc8a-428b-9a55-46d5551b57ba	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	286c9234-c533-4ef6-8774-a5de92fb8452	absent	\N	\N	\N
d92d4eaa-6b07-4d0c-9efa-b05b7fa14d47	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
c5e1b230-e3d2-4e3e-a808-bfd3a5f107bf	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
a897945e-2886-4c6e-8ba3-b8633b730bc6	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
a003cedb-d0a5-4321-97db-c195f78ad684	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
e0a72f1d-864d-406f-97cd-40d7492a1c77	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
96fa1785-bce4-469c-8e44-db1b4bd53bd1	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
22ca4da3-4004-4db7-a657-be6ee9f1ffc4	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
58c60ab8-e162-4b0a-ab49-119c09ea3f70	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
01db953a-4019-46d0-bd99-e6a1f78319b2	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
126a38b8-0ef1-493c-9906-0476739b9ae6	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	b2d261b4-3b33-4d88-9cd9-7cd634872290	absent	\N	\N	\N
1bc4f134-6842-4ec7-8c82-49792205e46b	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
bcae561c-6ef6-4872-9ef7-6b5412530f76	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
cee5087f-4cc7-4acb-a215-9f33e8d83993	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
11310932-0df8-4822-9a09-6f339f45405a	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
1e8d3aaa-2a05-46d7-9e71-c5f4ca33314f	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
25228daf-dc27-4ca5-88df-edc9637cc5c7	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
54023587-5698-4d94-8ee4-beee6ab8b51a	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
29405108-9b89-45d8-a04f-5fce5f2f966c	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
39faf47d-2ba9-4e01-b839-793e9cedbe5e	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
f3c3467b-4861-414a-8fd4-9478bbd60250	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
a642c00e-d387-4335-9e53-27ddd284fe3b	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	b2d261b4-3b33-4d88-9cd9-7cd634872290	late	\N	2025-04-27 02:30:43.243	kẹt xe
048ce5ee-6a84-4187-b127-f4194c5fd3f7	b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	04786799-ca23-4231-9cb1-527b8dd7e154	present	src/evidence_image/2025/04/24/b1b0c3cd-030e-4794-81e6-352e6b0a9f6c_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-24 09:28:25	\N
23b18bb0-1464-4f44-8586-1f169eefa731	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	absent	\N	\N	\N
eff91120-d7b3-4e0f-b931-5c11a685c9e9	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
ceb7eed6-3d3e-406f-8f9b-4016c83db2b0	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
35ded80a-d60c-4fed-b8dc-79e9adc91884	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
b118c83c-a785-4ff8-9fff-358a833331c4	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
97a71e49-bee3-46ac-ae15-26e8593a35f7	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
e4f22839-36c1-43ff-838b-68ba18ae7c29	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
b484bc01-d89c-4b2e-ba1f-e2edf033afab	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
07055815-2bef-4781-9cc4-43b74bdeb0b6	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
ecd9e3c6-752b-469a-ab7f-822cc01f06ef	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
fad37358-723e-4531-ba1c-6ad143bad7dc	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
8a152350-397a-4299-9454-cd29077298a7	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
50ca9fa8-8709-4215-ab15-a522478bd01e	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
6d2bfc86-baf5-49e3-ac1e-26df819d72e6	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
29a9b2ac-fe45-4000-a45c-015d71ccd8f5	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
e67375b0-f56e-44bf-89c9-9f4e10bf89e7	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
6004a60b-e8d0-4b83-a8c1-e691faa9a738	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
a247f89e-071c-4430-a32e-cc226ab40219	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
00be5aa1-1864-4cd2-b22a-ac7c12d1ffe2	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
722f1893-ae29-4480-842c-72288c3cc67c	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
6640172e-2e6a-45e3-84d9-74eb91cf28b7	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
d312cb8f-e60d-4c51-825d-cef919231f50	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
c6903252-2524-4144-8a96-37e50ce3cca5	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
cb1a9060-2b55-48d7-b4fa-eef24dd40efd	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
7bd9c67f-0562-4101-b113-f7d74e16a36b	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	04786799-ca23-4231-9cb1-527b8dd7e154	present	src/evidence_image/2025/04/25/a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537_04786799-ca23-4231-9cb1-527b8dd7e154.jpg	2025-04-25 14:35:23	\N
f3424636-034a-41cc-9ec7-df61308509b4	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	present	src/evidence_image/2025/04/25/a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537_0a7657d8-760e-40e1-8dcb-6ec8a56c8feb.jpg	2025-04-25 14:35:28	\N
f9d35de7-9b57-4860-99b0-aabe59ad07d5	a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	286c9234-c533-4ef6-8774-a5de92fb8452	present	src/evidence_image/2025/04/25/a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537_286c9234-c533-4ef6-8774-a5de92fb8452.jpg	2025-04-25 14:35:31	\N
3b32febc-c35e-4832-9cea-5ad73f200dc4	66ccdc85-ae89-414b-bb6f-2f6545444fe2	1dde1982-6705-4d6f-8f47-ddc1cfa83168	absent	\N	\N	\N
b91df77a-2757-4aa5-bc07-74ed43b4a348	66ccdc85-ae89-414b-bb6f-2f6545444fe2	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	absent	\N	\N	\N
e0811d70-de72-4b0d-a25e-5ff407cec68d	66ccdc85-ae89-414b-bb6f-2f6545444fe2	22309dfb-85ce-4221-beda-c12796788357	absent	\N	\N	\N
f6de554f-6e31-45b6-8108-8c0e495368e8	66ccdc85-ae89-414b-bb6f-2f6545444fe2	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	absent	\N	\N	\N
d27ffeb3-4c7c-4f92-8511-174a2afa5e34	66ccdc85-ae89-414b-bb6f-2f6545444fe2	29a519d0-5c29-4496-9687-1b7942dbd7ff	absent	\N	\N	\N
d74d3da1-9d41-421d-9364-cabd487685dd	66ccdc85-ae89-414b-bb6f-2f6545444fe2	3ef8aead-9239-43dc-9a7a-d9b3df374a99	absent	\N	\N	\N
f6afb510-dfff-4b97-8ca8-47eeed48ec7f	66ccdc85-ae89-414b-bb6f-2f6545444fe2	44bd3826-9511-411c-aa62-0b12e38584ed	absent	\N	\N	\N
38119e9a-1ce8-49ec-899d-2a7a83cae8b2	66ccdc85-ae89-414b-bb6f-2f6545444fe2	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	present	src/evidence_image/2025/04/26/66ccdc85-ae89-414b-bb6f-2f6545444fe2_0a7657d8-760e-40e1-8dcb-6ec8a56c8feb.jpg	2025-04-26 18:31:18	\N
f842bc97-a383-4ace-83e8-cbc2e4e04e03	66ccdc85-ae89-414b-bb6f-2f6545444fe2	286c9234-c533-4ef6-8774-a5de92fb8452	present	src/evidence_image/2025/04/26/66ccdc85-ae89-414b-bb6f-2f6545444fe2_286c9234-c533-4ef6-8774-a5de92fb8452.jpg	2025-04-26 18:31:21	\N
d035bd4d-a82c-45b7-842e-d22900b7ef1d	66ccdc85-ae89-414b-bb6f-2f6545444fe2	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	absent	\N	\N	\N
384483c7-941f-4c29-9c93-67d2e5095c5f	66ccdc85-ae89-414b-bb6f-2f6545444fe2	62e33c55-0d38-4a0b-87cc-468038adf30e	absent	\N	\N	\N
5dcc89a7-6251-4c87-95e6-8f20d434bea9	66ccdc85-ae89-414b-bb6f-2f6545444fe2	70c790b5-8ebd-4593-bf15-20bc404d408f	absent	\N	\N	\N
3b41dae2-2dcf-4daa-af2b-e2b4cfbc2bf3	66ccdc85-ae89-414b-bb6f-2f6545444fe2	821dee23-eaa0-447d-9aa3-f25b6570f98d	absent	\N	\N	\N
105c94b6-3b56-4b6b-a8b9-b0a9949d22d0	66ccdc85-ae89-414b-bb6f-2f6545444fe2	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	absent	\N	\N	\N
84799f73-b048-4ce9-9f33-e27f0b067df4	66ccdc85-ae89-414b-bb6f-2f6545444fe2	a416164c-78e4-46dd-8038-3c81b336190f	absent	\N	\N	\N
b9b10c40-33d8-4c7e-8ff7-04f48bed94d6	66ccdc85-ae89-414b-bb6f-2f6545444fe2	b6c748f3-d21c-48e4-a975-f604ebd9240a	absent	\N	\N	\N
36aca80f-3627-4ff2-a6b2-461846ba8cc7	66ccdc85-ae89-414b-bb6f-2f6545444fe2	bfea3fec-742d-4cab-abbc-a0fd90827cb1	absent	\N	\N	\N
140e57cd-640a-4563-bb99-050930a14578	66ccdc85-ae89-414b-bb6f-2f6545444fe2	bffc211b-88e3-4a54-9286-01511c608b0e	absent	\N	\N	\N
88fe6490-5449-409f-abfd-5fd3077b85b4	66ccdc85-ae89-414b-bb6f-2f6545444fe2	d9445cb8-2348-433e-ac8b-3af85feb6469	absent	\N	\N	\N
e0dff910-95f8-4ef4-a33e-19b517dc4005	66ccdc85-ae89-414b-bb6f-2f6545444fe2	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	absent	\N	\N	\N
f2d94a62-510c-40e4-9068-3ef84bf00baa	66ccdc85-ae89-414b-bb6f-2f6545444fe2	ff06e4d2-943b-4807-a3d7-395df26aa888	absent	\N	\N	\N
a60fbbc7-6c96-4fc3-828d-7bf72e3635f9	66ccdc85-ae89-414b-bb6f-2f6545444fe2	37a947b9-9a4b-4c99-908f-20aeb3010785	absent	\N	\N	\N
05da6981-3c03-4855-a1cc-779fbe095a87	66ccdc85-ae89-414b-bb6f-2f6545444fe2	d64d1501-42f4-4685-b7fe-1f9785f4ece8	absent	\N	\N	\N
6928dd44-58f1-4d61-beaa-90cd31a7227d	66ccdc85-ae89-414b-bb6f-2f6545444fe2	2e927240-2f96-4da4-9210-bc92b9ce7b8c	absent	\N	\N	\N
c34da4d3-cffb-4e53-af61-7017db85a270	66ccdc85-ae89-414b-bb6f-2f6545444fe2	874b8d9b-f901-4063-8b15-4305a35f708e	absent	\N	\N	\N
e00bc35b-7bd5-4c60-a4e4-136f67eee32d	66ccdc85-ae89-414b-bb6f-2f6545444fe2	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	late	\N	2025-04-26 16:22:35.247	
34f0874f-8de8-4a5d-a38f-4ed3439fbffe	66ccdc85-ae89-414b-bb6f-2f6545444fe2	b2d261b4-3b33-4d88-9cd9-7cd634872290	present	\N	2025-04-27 01:54:18.006	
9f4469c0-2621-4025-a0f0-1fca0edd4ad7	66ccdc85-ae89-414b-bb6f-2f6545444fe2	04786799-ca23-4231-9cb1-527b8dd7e154	absent	\N	2025-04-27 01:55:58.175	
\.


--
-- TOC entry 3813 (class 0 OID 16780)
-- Dependencies: 221
-- Data for Name: cameras; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.cameras (camera_id, classroom_id, camera_url, camera_type, location, created_at, socket_path) FROM stdin;
b6ac51dc-1a43-4e82-a039-ca96f88bddb9	4b26497e-7722-48e6-955c-f369cf16c411	http://192.168.0.100:81/stream	recognition	Gần cửa ra vào	2025-03-13 22:23:00.489712	0
a34800d4-2430-4ab8-8aaa-2568a58d6bc5	4b26497e-7722-48e6-955c-f369cf16c411	rtsp://admin:123456%40dlu@10.10.224.9:554/cam/realmonitor?channel=1&subtype=0	surveillance	Trên trần nhà bên trái	2025-03-24 13:40:08.051885	1
\.


--
-- TOC entry 3815 (class 0 OID 16812)
-- Dependencies: 223
-- Data for Name: class_students; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.class_students (class_id, student_id, status) FROM stdin;
c5ec083b-e4ed-40d3-8f5d-014d839a7617	f8455952-6f3f-485e-bd9b-942ce5bab472	active
e5f4d4bf-520b-4dbd-8cae-bded98d666c1	d64d1501-42f4-4685-b7fe-1f9785f4ece8	active
25f2ae91-3c2b-470d-823a-6097d9aca4fd	2e927240-2f96-4da4-9210-bc92b9ce7b8c	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	525fe628-c06f-413e-af2f-b421e5bdcc16	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	37a947b9-9a4b-4c99-908f-20aeb3010785	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	d64d1501-42f4-4685-b7fe-1f9785f4ece8	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	2e927240-2f96-4da4-9210-bc92b9ce7b8c	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	874b8d9b-f901-4063-8b15-4305a35f708e	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	286c9234-c533-4ef6-8774-a5de92fb8452	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	3ef8aead-9239-43dc-9a7a-d9b3df374a99	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	70c790b5-8ebd-4593-bf15-20bc404d408f	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	62e33c55-0d38-4a0b-87cc-468038adf30e	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	bffc211b-88e3-4a54-9286-01511c608b0e	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	821dee23-eaa0-447d-9aa3-f25b6570f98d	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	1dde1982-6705-4d6f-8f47-ddc1cfa83168	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	bfea3fec-742d-4cab-abbc-a0fd90827cb1	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	04786799-ca23-4231-9cb1-527b8dd7e154	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	b6c748f3-d21c-48e4-a975-f604ebd9240a	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	29a519d0-5c29-4496-9687-1b7942dbd7ff	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	b2d261b4-3b33-4d88-9cd9-7cd634872290	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	22309dfb-85ce-4221-beda-c12796788357	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	d9445cb8-2348-433e-ac8b-3af85feb6469	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	a416164c-78e4-46dd-8038-3c81b336190f	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	44bd3826-9511-411c-aa62-0b12e38584ed	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	04786799-ca23-4231-9cb1-527b8dd7e154	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	1dde1982-6705-4d6f-8f47-ddc1cfa83168	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	22309dfb-85ce-4221-beda-c12796788357	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	286c9234-c533-4ef6-8774-a5de92fb8452	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	29a519d0-5c29-4496-9687-1b7942dbd7ff	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	44bd3826-9511-411c-aa62-0b12e38584ed	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	821dee23-eaa0-447d-9aa3-f25b6570f98d	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	a416164c-78e4-46dd-8038-3c81b336190f	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	b2d261b4-3b33-4d88-9cd9-7cd634872290	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	b6c748f3-d21c-48e4-a975-f604ebd9240a	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	bfea3fec-742d-4cab-abbc-a0fd90827cb1	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	bffc211b-88e3-4a54-9286-01511c608b0e	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	d9445cb8-2348-433e-ac8b-3af85feb6469	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	ff06e4d2-943b-4807-a3d7-395df26aa888	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	37a947b9-9a4b-4c99-908f-20aeb3010785	active
c5ec083b-e4ed-40d3-8f5d-014d839a7617	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	banned
c5ec083b-e4ed-40d3-8f5d-014d839a7617	ff06e4d2-943b-4807-a3d7-395df26aa888	inactive
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	2e927240-2f96-4da4-9210-bc92b9ce7b8c	inactive
25f2ae91-3c2b-470d-823a-6097d9aca4fd	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	1dde1982-6705-4d6f-8f47-ddc1cfa83168	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	22309dfb-85ce-4221-beda-c12796788357	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	286c9234-c533-4ef6-8774-a5de92fb8452	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	29a519d0-5c29-4496-9687-1b7942dbd7ff	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	3ef8aead-9239-43dc-9a7a-d9b3df374a99	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	44bd3826-9511-411c-aa62-0b12e38584ed	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	62e33c55-0d38-4a0b-87cc-468038adf30e	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	70c790b5-8ebd-4593-bf15-20bc404d408f	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	821dee23-eaa0-447d-9aa3-f25b6570f98d	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	874b8d9b-f901-4063-8b15-4305a35f708e	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	a416164c-78e4-46dd-8038-3c81b336190f	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	b2d261b4-3b33-4d88-9cd9-7cd634872290	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	b6c748f3-d21c-48e4-a975-f604ebd9240a	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	bfea3fec-742d-4cab-abbc-a0fd90827cb1	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	bffc211b-88e3-4a54-9286-01511c608b0e	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	d64d1501-42f4-4685-b7fe-1f9785f4ece8	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	d9445cb8-2348-433e-ac8b-3af85feb6469	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	525fe628-c06f-413e-af2f-b421e5bdcc16	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	ff06e4d2-943b-4807-a3d7-395df26aa888	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	37a947b9-9a4b-4c99-908f-20aeb3010785	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	04786799-ca23-4231-9cb1-527b8dd7e154	
25f2ae91-3c2b-470d-823a-6097d9aca4fd	f8455952-6f3f-485e-bd9b-942ce5bab472	
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	25b90d1d-4e1d-48c8-adcb-a334c517fc2d	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	286c9234-c533-4ef6-8774-a5de92fb8452	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	29a519d0-5c29-4496-9687-1b7942dbd7ff	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	3ef8aead-9239-43dc-9a7a-d9b3df374a99	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	44bd3826-9511-411c-aa62-0b12e38584ed	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	50acc0da-9191-4e0d-b2f1-f95f49e85c8a	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	62e33c55-0d38-4a0b-87cc-468038adf30e	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	70c790b5-8ebd-4593-bf15-20bc404d408f	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	821dee23-eaa0-447d-9aa3-f25b6570f98d	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	874b8d9b-f901-4063-8b15-4305a35f708e	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	9977e3f2-a43f-4e5a-9115-6ba44bc54cec	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	a416164c-78e4-46dd-8038-3c81b336190f	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	b2d261b4-3b33-4d88-9cd9-7cd634872290	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	b6c748f3-d21c-48e4-a975-f604ebd9240a	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	bfea3fec-742d-4cab-abbc-a0fd90827cb1	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	bffc211b-88e3-4a54-9286-01511c608b0e	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	d64d1501-42f4-4685-b7fe-1f9785f4ece8	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	d9445cb8-2348-433e-ac8b-3af85feb6469	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	525fe628-c06f-413e-af2f-b421e5bdcc16	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	ff06e4d2-943b-4807-a3d7-395df26aa888	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	37a947b9-9a4b-4c99-908f-20aeb3010785	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	2e927240-2f96-4da4-9210-bc92b9ce7b8c	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	04786799-ca23-4231-9cb1-527b8dd7e154	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	f8455952-6f3f-485e-bd9b-942ce5bab472	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	874b8d9b-f901-4063-8b15-4305a35f708e	banned
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	0ff33482-fc8b-4a12-b6e8-5ed58070d21e	banned
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	1dde1982-6705-4d6f-8f47-ddc1cfa83168	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	1eac1149-9c04-4934-93f9-74f2e7fc7fd3	active
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	22309dfb-85ce-4221-beda-c12796788357	active
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	3ef8aead-9239-43dc-9a7a-d9b3df374a99	banned
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	banned
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	70c790b5-8ebd-4593-bf15-20bc404d408f	inactive
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	62e33c55-0d38-4a0b-87cc-468038adf30e	inactive
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	d64d1501-42f4-4685-b7fe-1f9785f4ece8	active
\.


--
-- TOC entry 3814 (class 0 OID 16795)
-- Dependencies: 222
-- Data for Name: classes; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.classes (class_id, class_name, course_id, lecturer_id, created_at, current_lesson) FROM stdin;
0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	CTk45A	08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:42:02.855797+07	0
25f2ae91-3c2b-470d-823a-6097d9aca4fd	CTK45B	08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:42:02.855797+07	0
c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	CTK45A	dd54afe2-8b65-49c6-ad6c-3aaa2f727eb9	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:42:02.855797+07	0
e5f4d4bf-520b-4dbd-8cae-bded98d666c1	CTK46B	3bcbf8c5-3056-49ff-a01a-e76853ccb116	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:42:02.855797+07	0
c5ec083b-e4ed-40d3-8f5d-014d839a7617	CTK45A	3bcbf8c5-3056-49ff-a01a-e76853ccb116	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:42:02.855797+07	0
\.


--
-- TOC entry 3812 (class 0 OID 16769)
-- Dependencies: 220
-- Data for Name: classrooms; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.classrooms (classroom_id, room_name, room_type, location, description, created_at) FROM stdin;
ec997950-900c-4686-a7b5-ccfead53caef	A24 Lau	Computer lab	Tòa nhà A24, phòng tầng lầu 	Description for Classroom 	2025-03-13 08:28:27.136532
4b26497e-7722-48e6-955c-f369cf16c411	A24.1	Computer lab	Tòa nhà thư viện, phòng thư viện 3 	Description for Classroom 	2025-03-13 08:29:34.027998
dcaa7ef5-2946-469b-8e32-55c394dce285	TV2	Computer lab	Tòa nhà A24, phòng tầng trệt 	Description for Classroom 	2025-03-13 08:28:07.92423
\.


--
-- TOC entry 3811 (class 0 OID 16757)
-- Dependencies: 219
-- Data for Name: courses; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.courses (course_id, course_name, main_lecturer_id, created_at, total_lesson, semester_id) FROM stdin;
08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a	Cấu trúc dữ liệu giải thuật	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:41:22.933912+07	60	08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a
9fe6aa2c-4db6-447b-bcd3-9ce2fc572d11	Lập trình nhúng	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:41:22.933912+07	60	08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a
e93f2b9b-3cd4-4bfe-8325-0184b39e496f	Quy trình phát triển phần mềm	850a0e1c-3107-454f-9d46-211a1a92491b	2025-03-13 08:41:22.933912+07	60	08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a
dd54afe2-8b65-49c6-ad6c-3aaa2f727eb9	IoT vạn vật	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:41:22.933912+07	60	3bcbf8c5-3056-49ff-a01a-e76853ccb116
3bcbf8c5-3056-49ff-a01a-e76853ccb116	Lập trình hướng đối tượng	2d536da8-fdf3-437b-a812-fb4e08aad955	2025-03-13 08:41:22.933912+07	60	3bcbf8c5-3056-49ff-a01a-e76853ccb116
\.


--
-- TOC entry 3810 (class 0 OID 16743)
-- Dependencies: 218
-- Data for Name: lecturers; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.lecturers (lecturer_id, lecturer_code, face_embedding, lectainer_code) FROM stdin;
2d536da8-fdf3-437b-a812-fb4e08aad955	Nguyễn Văn A	[0.417628,0.44639108,0.33714303,0.036381483,0.5042452,0.8371904,0.30427292,0.49703032,0.5321601,0.6978969,0.14144109,0.5524582,0.67466676,0.90741545,0.29606995,0.32755002,0.088773474,0.64103246,0.86271614,0.3745927,0.9845358,0.094674274,0.6658895,0.77799225,0.12218146,0.1925155,0.69465286,0.35524106,0.8780045,0.15422879,0.025592389,0.051688276,0.13019155,0.07947123,0.687093,0.21014273,0.3209803,0.19577771,0.6083054,0.21272017,0.6229169,0.72709256,0.518653,0.20815603,0.6643742,0.50866187,0.45096135,0.8264739,0.31135347,0.82560694,0.68449014,0.3650091,0.17144163,0.7100849,0.82270813,0.30835238,0.033182275,0.9162566,0.006754369,0.1624152,0.9792352,0.6636346,0.15080085,0.33315575,0.48667696,0.8174427,0.35539716,0.73602307,0.3547099,0.71949387,0.30761364,0.4447206,0.51364106,0.3053221,0.7641938,0.062378585,0.88389915,0.83495,0.23826589,0.6591297,0.531947,0.18445821,0.17150164,0.89504987,0.6969479,0.11641483,0.8775123,0.7028742,0.7148675,0.8881576,0.9545169,0.9576974,0.6137353,0.83417714,0.878015,0.5363688,0.26288295,0.11098652,0.8997177,0.34377736,0.40603262,0.5949322,0.31339648,0.4342297,0.6148152,0.9296626,0.16305844,0.027698945,0.7635282,0.79873484,0.47137144,0.58862966,0.17454816,0.003929169,0.673029,0.3678769,0.8782158,0.33493385,0.2131196,0.30844575,0.6660635,0.012877181,0.09680338,0.900798,0.09482156,0.97634363,0.19237907,0.047534734,0.9693491,0.8241341,0.1682528,0.92544854,0.23303238,0.09054317,0.056951158,0.1304847,0.08343046,0.29241258,0.63210624,0.21937172,0.4927533,0.16356964,0.47717088,0.773843,0.48199305,0.43315578,0.879637,0.2986009,0.14825982,0.3986416,0.4456688,0.45561573,0.8563687,0.49015763,0.8434052,0.54988587,0.78527987,0.64281946,0.48675194,0.17035966,0.3783089,0.46786585,0.21960351,0.36102617,0.5488661,0.36745352,0.23098995,0.049550407,0.15475844,0.98641425,0.40075886,0.6671126,0.043181658,0.5519068,0.98297703,0.019931624,0.4663176,0.2033929,0.62559915,0.06403821,0.19644785,0.22206801,0.15999122,0.33584082,0.7217058,0.38890222,0.63108623,0.46441492,0.5331108,0.04798477,0.2691842,0.1102235,0.35740495,0.8144399,0.21340886,0.4386973,0.029579094,0.6888106,0.28004125,0.97604305,0.5384647,0.9167726,0.78919643,0.11688035,0.047193274,0.37839815,0.7273831,0.5253342,0.51697725,0.6339206,0.3540717,0.5169651,0.13959852,0.97796845,0.056268822,0.64974356,0.6816385,0.5179227,0.16596751,0.19695252,0.27626458,0.4374178,0.59622514,0.600203,0.10679019,0.3510156,0.47366327,0.86492604,0.44902676,0.2776805,0.7787219,0.39693984,0.38063583,0.484025,0.7015477,0.013553595,0.38903895,0.25451878,0.5563457,0.3097521,0.15414783,0.02036593,0.47729495,0.80006945,0.91331,0.34994096,0.2069317,0.082833946,0.07267296,0.3599,0.32786155,0.77177024,0.045845475,0.60443985,0.013292884,0.89748996,0.9696687,0.25274435,0.3501973,0.5831357,0.7488363,0.79529667,0.068163544,0.071561866,0.7151924,0.50769156,0.6261158,0.0130970795,0.294296,0.70386374,0.4182324,0.45309973,0.61493915,0.7642766,0.6166018,0.3010543,0.3520475,0.88562703,0.16390601,0.7804784,0.07156822,0.49636313,0.111829825,0.081071936,0.15127327,0.9813992,0.8309223,0.9367723,0.9179213,0.16190033,0.6386984,0.57534224,0.70399326,0.61503446,0.66232353,0.994003,0.33602732,0.51726526,0.6487118,0.8944501,0.72023576,0.7893385,0.76879483,0.77972513,0.74748254,0.9920518,0.9257281,0.87967646,0.13955358,0.8466108,0.18571189,0.24280196,0.8039244,0.26612183,0.20446077,0.18903342,0.15186454,0.17439117,0.48189068,0.9248244,0.024663573,0.36496627,0.32629457,0.09293525,0.7928172,0.023376333,0.18137312,0.4890023,0.3854142,0.7122289,0.17109208,0.842622,0.24364613,0.1806812,0.598141,0.35095376,0.8338663,0.23584844,0.9116169,0.5653668,0.3996415,0.98180884,0.55092424,0.7496603,0.13358545,0.36096653,0.9169048,0.2015966,0.16118133,0.1519254,0.8029371,0.2961486,0.57721984,0.4934933,0.3575579,0.66144633,0.93766415,0.8978622,0.2956468,0.6339994,0.79953116,0.037219565,0.015127186,0.068880826,0.7735065,0.49371606,0.2314114,0.15549113,0.59432316,0.36019906,0.33666667,0.7887984,0.7434118,0.012166922,0.6146337,0.89952415,0.23552373,0.36618024,0.23476185,0.6675774,0.90913117,0.48114556,0.9919279,0.08271962,0.4276066,0.23693646,0.69517267,0.23652838,0.34152672,0.14760336,0.39248005,0.57077914,0.53378296,0.49075592,0.6954951,0.37386438,0.2317241,0.5166967,0.8693213,0.003157413,0.15825146,0.96649826,0.5087255,0.62486935,0.04633884,0.8610339,0.58344656,0.2572469,0.4432989,0.9545507,0.6815643,0.6740519,0.25503847,0.59603125,0.37470958,0.96672434,0.082483836,0.25598642,0.44437143,0.5567846,0.040490095,0.16229445,0.85378975,0.42426556,0.7857744,0.2944446,0.060376514,0.9597437,0.65633744,0.7827681,0.021020938,0.6081312,0.49822295,0.67520624,0.49918053,0.014871374,0.2513091,0.17770079,0.68559474,0.08499928,0.31824142,0.5980462,0.7053185,0.60482013,0.6598355,0.11857297,0.8790968,0.40963137,0.41585532,0.41344854,0.46039942,0.09867195,0.69720095,0.112808496,0.096164495,0.61615425,0.38499293,0.62453675,0.53366053,0.59554756,0.19405758,0.6119739,0.9782084,0.16520941,0.49772114,0.9930865,0.9609703,0.22708358,0.36068588,0.6464241,0.7968983,0.05768156,0.301192,0.4814044,0.30713627,0.7130901,0.7253858,0.9209676,0.5732045,0.066306844,0.5262149,0.8204797,0.5036503,0.33499226,0.7376532,0.6740041,0.65730363,0.9160884,0.9257914,0.5977888,0.9744103,0.42067996,0.45273826,0.6330071,0.5797792,0.6373643,0.8335185,0.7114136,0.111604825,0.8969328,0.7672872,0.9026182,0.9682708,0.037327822,0.4058095,0.6407147,0.3894563,0.34608346,0.5155247,0.94047046,0.7777744,0.11792905]	\N
5f7d2b42-0198-4871-b364-f9c83a34bd27	Trần Văn B	[0.7569973,0.6888395,0.3554259,0.5810818,0.82933235,0.21923289,0.7035089,0.23998742,0.7583787,0.2914827,0.4166154,0.7874758,0.58563685,0.65574676,0.49695903,0.85588795,0.42471856,0.8381285,0.436525,0.66231704,0.4969917,0.95349306,0.471464,0.6802503,0.7503444,0.67878234,0.53433573,0.9734333,0.36977938,0.46912962,0.38482502,0.074916,0.0098402705,0.8739326,0.4428235,0.60550433,0.6725852,0.58975244,0.096383296,0.5092813,0.7366518,0.6636845,0.28472313,0.25067315,0.44881168,0.8820319,0.8755755,0.05771287,0.79357153,0.527704,0.23983721,0.6413335,0.9338092,0.79595447,0.98958683,0.52689826,0.5223032,0.6051336,0.47073704,0.61898583,0.86343116,0.9629479,0.79171866,0.63837403,0.75116134,0.14183974,0.8397175,0.9383833,0.5661323,0.8345686,0.14944376,0.20177601,0.32938847,0.36317655,0.052218378,0.26440665,0.30340436,0.5903741,0.886784,0.15919852,0.04204193,0.020178977,0.4132941,0.7845312,0.94578373,0.21818167,0.6069482,0.7003342,0.29008693,0.23489022,0.002123369,0.5646144,0.91571313,0.04158126,0.13770422,0.71371937,0.87765306,0.87198275,0.39522645,0.12612844,0.46876526,0.8208033,0.6161464,0.44396552,0.5791552,0.44133726,0.9415646,0.8733862,0.53331876,0.9098501,0.50519896,0.7456898,0.48131198,0.30388626,0.31847632,0.45793423,0.7701703,0.69682986,0.24496579,0.6017827,0.027401095,0.12016414,0.5177493,0.472304,0.3790934,0.56708705,0.67907363,0.86381346,0.6145922,0.46970752,0.7218467,0.25715885,0.38310438,0.559497,0.33894244,0.49983013,0.9229947,0.5409703,0.9523178,0.29452214,0.6857373,0.81015134,0.030617159,0.11547717,0.94456106,0.98452085,0.24752116,0.12115856,0.2434564,0.49851948,0.37759322,0.54551303,0.42008364,0.34435427,0.9289653,0.04668265,0.32013655,0.2782529,0.22417645,0.67805016,0.9405507,0.60463685,0.74848014,0.49476412,0.11833615,0.88216,0.32454443,0.19761577,0.3197719,0.5745602,0.5928866,0.81410855,0.46651566,0.49750495,0.41408628,0.060448892,0.95706147,0.65463096,0.56800187,0.37199014,0.61080235,0.68021095,0.84994376,0.20232262,0.76159,0.26688865,0.40649852,0.43605548,0.9829319,0.5916583,0.17797956,0.89251345,0.651798,0.6945595,0.3125731,0.6975011,0.5228318,0.7795857,0.070980646,0.08478771,0.28069955,0.7451398,0.12195637,0.6895489,0.8069305,0.30040994,0.33031663,0.7412588,0.7854449,0.64204514,0.7123329,0.95414287,0.3321825,0.9527617,0.39729613,0.7904074,0.9493144,0.8286868,0.68253404,0.828335,0.8387309,0.046213247,0.038892902,0.6972422,0.8334865,0.13068601,0.3250529,0.42742378,0.7033375,0.25945732,0.9872402,0.67981344,0.34529394,0.5268433,0.5808751,0.44520923,0.104882866,0.9408774,0.47269845,0.91839665,0.68929446,0.008147508,0.9308582,0.12686776,0.46396643,0.56887996,0.652077,0.3442373,0.7068448,0.99959326,0.6361597,0.0053264126,0.5016915,0.13456556,0.14674945,0.11240904,0.966971,0.075141124,0.58746076,0.5553583,0.7124658,0.08908897,0.33735484,0.6350713,0.6063722,0.032312747,0.13321476,0.7321945,0.9089917,0.031360514,0.42704666,0.26014465,0.9025829,0.659073,0.31690624,0.18231538,0.9185082,0.005964408,0.5588475,0.24901065,0.17943247,0.21309143,0.011525505,0.7996456,0.4214796,0.04951848,0.84184307,0.13202438,0.49297407,0.55997086,0.3485406,0.08436818,0.48097885,0.6393336,0.7316197,0.52923524,0.30839562,0.5845619,0.8410248,0.34559855,0.7846583,0.5921034,0.83387977,0.90510225,0.15552935,0.87513626,0.34039855,0.22647786,0.8248922,0.5804237,0.6165499,0.22559832,0.21087098,0.53942156,0.39565128,0.51232773,0.69853586,0.96543205,0.2844767,0.06827967,0.30281547,0.68673223,0.27728546,0.115347035,0.53804195,0.86406374,0.5943466,0.7078135,0.6380026,0.7790364,0.40232232,0.24424219,0.0985251,0.31321064,0.9813735,0.4511793,0.06548348,0.3717109,0.07466437,0.81398606,0.9145186,0.5089976,0.822681,0.8367917,0.19253725,0.9490261,0.6868111,0.25326258,0.6300104,0.13495415,0.6105685,0.5188432,0.25303677,0.420366,0.32303125,0.9952175,0.13336213,0.91385156,0.86255157,0.09583212,0.59050447,0.64524305,0.39574224,0.6607487,0.3195688,0.8833094,0.13351947,0.42529646,0.9105859,0.9659475,0.8473507,0.7904182,0.42493048,0.18636851,0.33077154,0.42546588,0.5190642,0.49454296,0.74655,0.0046317,0.52889097,0.6673218,0.8327343,0.90631515,0.967848,0.6676358,0.9293047,0.96207035,0.7509401,0.16209386,0.22730346,0.8186375,0.5620918,0.622153,0.7236206,0.25551987,0.9273208,0.029111296,0.33330926,0.45384794,0.9542885,2.1197784e-05,0.799014,0.22722782,0.23915078,0.20313388,0.8415458,0.96177566,0.07848578,0.9773902,0.2383741,0.7877204,0.47407055,0.7071564,0.6650836,0.26918808,0.5894358,0.027324049,0.63096786,0.4080348,0.767222,0.61146784,0.39325362,0.11821398,0.50599235,0.023772981,0.16133392,0.6985247,0.7550746,0.597794,0.010923038,0.6550599,0.63853514,0.8688042,0.70271915,0.19077317,0.49844074,0.00026451983,0.2638539,0.9301093,0.8281096,0.70106995,0.35135752,0.89096713,0.65024304,0.94730943,0.96606195,0.7216552,0.4720859,0.8668978,0.11377739,0.8835443,0.5801266,0.87153524,0.5025134,0.36799538,0.8168855,0.5834952,0.11937307,0.43939078,0.8186357,0.80978775,0.0026661411,0.10227669,0.33092114,0.3292235,0.7218534,0.8174584,0.3366962,0.62453324,0.42129374,0.61875415,0.062265135,0.2849045,0.46721345,0.44518572,0.90528595,0.3085189,0.8931913,0.68862873,0.43493375,0.74789006,0.26253653,0.958526,0.06555165,0.3366881,0.7346601,0.10915938,0.72801137,0.19253029,0.1740125,0.9892233,0.37102604,0.008372305,0.8056714,0.63322306,0.080420494,0.4858537,0.8974492,0.25644523,0.106772125,0.84789085,0.03917717,0.73944134,0.057205703,0.99252933,0.8086881,0.08010156,0.7443225,0.20895484,0.51276267,0.96092254]	\N
850a0e1c-3107-454f-9d46-211a1a92491b	Tăng C	[0.09019595,0.6274683,0.6231904,0.6175029,0.7909441,0.47123766,0.52900356,0.34715495,0.74532974,0.39944327,0.5964127,0.1371796,0.3819915,0.6365388,0.45347923,0.6897049,0.70936775,0.100347914,0.07010496,0.85026014,0.9867766,0.22146554,0.99166334,0.60714567,0.11799709,0.04924974,0.254655,0.69962376,0.22358115,0.75219536,0.20421048,0.3305204,0.38538226,0.98736703,0.029083844,0.9302774,0.0056477515,0.33872434,0.35445198,0.5763784,0.7475783,0.9182124,0.15348852,0.060189307,0.47614086,0.3388985,0.76307184,0.56260663,0.54948616,0.8266499,0.1082744,0.37316832,0.6138177,0.6702806,0.8810168,0.18413654,0.32866967,0.07687012,0.5312155,0.2702474,0.79849166,0.0027053612,0.02669612,0.7142082,0.020565402,0.9700019,0.31358436,0.3437259,0.8948493,0.88249624,0.7077321,0.9353343,0.53918403,0.84126776,0.025742438,0.9261056,0.8944749,0.43414053,0.8696621,0.009572608,0.3946836,0.114156045,0.21071702,0.5926339,0.9328305,0.45580393,0.5660113,0.027394501,0.23982374,0.59862417,0.61269,0.92445654,0.7466868,0.31281012,0.2976144,0.45210868,0.4067802,0.75160867,0.8757306,0.44939288,0.85666496,0.8144268,0.5793971,0.20731394,0.7568947,0.68883467,0.009414652,0.46742964,0.9126517,0.9758526,0.13663158,0.59736586,0.16499795,0.3557951,0.6401092,0.15441363,0.28513733,0.60264456,0.31840527,0.45331952,0.61653525,0.8257346,0.84471405,0.3369739,0.7189818,0.010227846,0.22339928,0.06951939,0.14252067,0.056364458,0.9957031,0.09555536,0.5720792,0.34517878,0.5646167,0.8621707,0.38423046,0.25676715,0.2781218,0.7018174,0.8076228,0.08294248,0.65425044,0.67469746,0.1677762,0.6069155,0.36847308,0.4036803,0.5576384,0.08792497,0.8256117,0.96781844,0.9987135,0.8464474,0.013807602,0.9914281,0.7128205,0.71167463,0.7661595,0.3980908,0.3568017,0.80468255,0.35668358,0.84773827,0.7102152,0.66137946,0.8239501,0.7115816,0.94556963,0.79322225,0.7119584,0.4464716,0.8565209,0.83809453,0.42441717,0.8354011,0.607483,0.61162245,0.37288415,0.90805346,0.72202927,0.41881925,0.61512595,0.56194746,0.7187353,0.74439794,0.6272197,0.4355455,0.6983741,0.31135032,0.68727,0.7591441,0.027602162,0.6416152,0.36366364,0.81654155,0.27633017,0.029197553,0.3988081,0.6064588,0.8583162,0.10889384,0.0058302903,0.5065572,0.19272283,0.44501993,0.84874475,0.04675699,0.97186846,0.7099737,0.94675547,0.5959981,0.7665716,0.65101194,0.9306661,0.15876961,0.23959832,0.70600915,0.6726801,0.82691944,0.8320146,0.44126493,0.3731221,0.18886447,0.2162726,0.7853448,0.6759287,0.040716574,0.8514894,0.71904373,0.800086,0.39416832,0.079465166,0.26663464,0.64994335,0.7244577,0.84927964,0.17720422,0.30526805,0.6017249,0.2483585,0.24822497,0.47854778,0.18186973,0.5682228,0.9289847,0.12227969,0.38009682,0.5756716,0.7346349,0.048906945,0.67419416,0.9355826,0.17856683,0.019900056,0.9117348,0.16690725,0.5396453,0.5296564,0.78606147,0.7823222,0.39227676,0.93975055,0.5406709,0.73506933,0.23159249,0.7495543,0.5638185,0.03257298,0.087295845,0.029313803,0.69321376,0.16436207,0.665441,0.13829172,0.97052056,0.79277253,0.29937255,0.5075885,0.9928798,0.46995196,0.42323813,0.43473724,0.8139297,0.118123636,0.24428453,0.08711385,0.5067178,0.6460268,0.23199883,0.5444166,0.076012515,0.9241005,0.05579461,0.13953502,0.31007865,0.75712186,0.043462455,0.79860836,0.6732736,0.04228709,0.8579437,0.63202024,0.06848919,0.6316131,0.67703146,0.4932467,0.589399,0.38538012,0.4570425,0.9752529,0.43649715,0.56311876,0.6572065,0.9012559,0.87829906,0.7834321,0.021652643,0.6484561,0.9984584,0.7993663,0.81380254,0.26150754,0.8501488,0.11288859,0.9240058,0.571672,0.55407035,0.56643724,0.31707484,0.92168593,0.99297917,0.867669,0.60100824,0.07041532,0.64195806,0.43081802,0.50729203,0.89307606,0.29966047,0.9016151,0.21152437,0.7697812,0.5155442,0.28822747,0.5565115,0.56597155,0.7534656,0.3291032,0.8616108,0.10525896,0.9831033,0.09744246,0.36385575,0.3205004,0.30749804,0.6574916,0.29001456,0.32757592,0.78849584,0.6110614,0.48010093,0.4124661,0.54303336,0.17224976,0.12255703,0.26996592,0.8308745,0.8747424,0.5047776,0.792402,0.081467435,0.47542995,0.85195446,0.6044624,0.48804924,0.18468864,0.8065608,0.5744864,0.9207716,0.097176164,0.94857806,0.332932,0.6996658,0.93706805,0.62528855,0.7093456,0.84751475,0.7292405,0.7928506,0.5033906,0.20539439,0.34316692,0.5724091,0.7209037,0.63515425,0.9222308,0.8328022,0.8096666,0.83135736,0.54607904,0.03258283,0.42787552,0.9743714,0.4075873,0.83658874,0.19619292,0.81587654,0.9295679,0.7355876,0.29579952,0.037636045,0.053680956,0.5820597,0.19745101,0.1933106,0.3889188,0.7954128,0.20396447,0.110478394,0.8260245,0.4047791,0.84374565,0.9450493,0.9500869,0.12573153,0.33399177,0.9829144,0.76666766,0.45537466,0.5621561,0.8301864,0.7266121,0.6558739,0.86893415,0.37712997,0.47810206,0.260433,0.7216683,0.4082882,0.37065402,0.15696855,0.044701196,0.6525667,0.19709979,0.893726,0.58122104,0.55490655,0.21430689,0.13463305,0.6069245,0.9526883,0.93573123,0.17648451,0.58460414,0.23461777,0.5015702,0.9010071,0.17145202,0.7613886,0.40339184,0.9155203,0.39146945,0.28931263,0.97546583,0.12966864,0.8532666,0.68486756,0.9671737,0.53448725,0.386822,0.97166884,0.82831615,0.42581078,0.9049101,0.034850568,0.7271341,0.2897941,0.1125747,0.14932783,0.19887319,0.41787732,0.2151559,0.6639346,0.62339836,0.6903106,0.9837623,0.61211914,0.5969197,0.0920208,0.42962858,0.16278312,0.18739842,0.7192636,0.61221516,0.7408974,0.62839115,0.47381437,0.628584,0.6834703,0.013394051,0.26132017,0.58191127,0.8907064,0.89974827,0.95912534,0.8467434,0.48020336,0.6154693,0.9134266,0.3484158,0.3834381]	\N
98b24b88-7c84-4d9e-b8a9-35060056f170	Trần D	[0.10425946,0.37734395,0.6794533,0.7846997,0.69512033,0.6825708,0.0035934146,0.69018316,0.6459069,0.41821185,0.47400495,0.71395093,0.4675697,0.7406303,0.14402261,0.9831573,0.16783518,0.56542474,0.56812716,0.15583795,0.531686,0.095763244,0.035449084,0.5210409,0.50041103,0.66638315,0.27483565,0.2234237,0.8431868,0.036327958,0.31654465,0.30730549,0.14334123,0.52602774,0.58398974,0.35741654,0.21561429,0.9956772,0.59665453,0.882733,0.7615989,0.4125683,0.052416712,0.6072498,0.086946346,0.67908454,0.641583,0.58529866,0.31032366,0.57617927,0.28744835,0.75993264,0.40357742,0.40245962,0.028775271,0.6454369,0.2990083,0.64730316,0.38771662,0.2310522,0.40442926,0.9905807,0.73557645,0.050823182,0.37126678,0.8021169,0.8238919,0.54163194,0.41968188,0.15574792,0.46104756,0.61519563,0.07530819,0.8437955,0.7638942,0.47840697,0.02168194,0.04978925,0.50813174,0.23295334,0.094474465,0.17459458,0.2945949,0.79488224,0.5953306,0.23320146,0.7252131,0.35300478,0.6684207,0.87504095,0.6769242,0.9958542,0.13243873,0.67465633,0.12856431,0.77062005,0.519475,0.15252483,0.63304245,0.6301261,0.72818005,0.6512727,0.24235915,0.31213182,0.20926292,0.9206483,0.9146796,0.25918236,0.309753,0.8599795,0.8773615,0.8800378,0.57922846,0.76992697,0.3129867,0.103718616,0.014206672,0.4553716,0.24223821,0.5681614,0.10243368,0.35957024,0.8315477,0.78796154,0.17101455,0.5078886,0.958872,0.16249229,0.14012454,0.42835003,0.5121215,0.36855286,0.16732642,0.93055737,0.6644705,0.08174763,0.07784589,0.51241577,0.21470925,0.56893444,0.02921196,0.2719379,0.6143316,0.444934,0.093032725,0.17702661,0.6485939,0.96194017,0.664003,0.079439946,0.5220219,0.85416573,0.72263014,0.0427518,0.24992482,0.36481756,0.6019361,0.66234773,0.52118784,0.8595403,0.043165006,0.40726805,0.094368204,0.070941254,0.9350519,0.66759515,0.23172757,0.8238884,0.37427196,0.24351123,0.26624864,0.28186682,0.3887364,0.6729034,0.50657433,0.24022037,0.19927353,0.5284493,0.07542547,0.040286854,0.5101098,0.50766134,0.33376607,0.7657833,0.7495085,0.33608586,0.28116155,0.27006584,0.6814897,0.6518491,0.59055597,0.77876335,0.57249826,0.19353619,0.9406523,0.21916945,0.95397574,0.8272331,0.3071454,0.17704725,0.80847085,0.025430243,0.8076888,0.4501167,0.50902826,0.46633208,0.19449842,0.1755889,0.44803077,0.74418646,0.23103808,0.028182335,0.5658129,0.5398261,0.50037056,0.812097,0.29349497,0.7309247,0.5666839,0.30624247,0.14109483,0.3995165,0.93655086,0.29140243,0.6420609,0.98556244,0.59668165,0.27550378,0.38551182,0.98704785,0.076506086,0.40414685,0.2891997,0.3986465,0.57069814,0.5015963,0.5735943,0.03506924,0.48962268,0.44308826,0.92259115,0.39244974,0.16492148,0.29254332,0.02887616,0.6510721,0.91517526,0.52979577,0.11564417,0.24222593,0.19484511,0.48843005,0.476783,0.50390613,0.67972046,0.21556513,0.07079957,0.047781304,0.72947896,0.40081182,0.1303265,0.94793427,0.83118606,0.6116647,0.14839521,0.19150348,0.9913226,0.67186475,0.8018501,0.9770753,0.013058964,0.5289673,0.2025489,0.5541254,0.5469761,0.7862529,0.21559666,0.7578868,0.16330108,0.57781076,0.16248412,0.1559867,0.23694688,0.5206264,0.95403177,0.08766633,0.06681523,0.925074,0.868848,0.5764438,0.9865634,0.5212402,0.99587333,0.24727619,0.5150721,0.4423014,0.51518506,0.60074115,0.67172533,0.21171507,0.39723495,0.049107067,0.28148034,0.1390585,0.98249644,0.67241895,0.42904976,0.4942277,0.27270943,0.5956572,0.8231293,0.018515345,0.58453256,0.06871355,0.9195982,0.22245863,0.23314026,0.84038025,0.8469562,0.72275996,0.7030755,0.49577114,0.96066976,0.116755754,0.35129088,0.8775834,0.024709184,0.13516992,0.27380663,0.6967666,0.91644084,0.64649075,0.0279667,0.13543108,0.32392573,0.99431586,0.52774537,0.89557713,0.46347427,0.16240059,0.08895621,0.37633163,0.42327082,0.3990315,0.9988493,0.0736647,0.27756,0.6563699,0.9945863,0.9418005,0.7945702,0.08513268,0.13500696,0.7877533,0.21743493,0.22802101,0.63435316,0.37837833,0.6335196,0.08575163,0.6167006,0.28494182,0.32331908,0.6120571,0.8011909,0.67237705,0.51412433,0.49164894,0.9942295,0.81498694,0.8083634,0.7139634,0.27719757,0.42592537,0.30537075,0.35229364,0.14334127,0.19540857,0.4011761,0.22717316,0.6871285,0.37582615,0.32989374,0.03998336,0.41981083,0.333482,0.36884513,0.94412416,0.14573579,0.66282606,0.4040333,0.14457145,0.309547,0.0399831,0.44424838,0.9530053,0.7468609,0.7498665,0.746975,0.37514222,0.38287988,0.53865623,0.3142349,0.30693424,0.9789452,0.30675945,0.9847148,0.056754485,0.8470176,0.30838126,0.99523044,0.21594487,0.4835765,0.23817353,0.91294235,0.57119966,0.9752333,0.41355655,0.355872,0.80295897,0.1687635,0.17901674,0.64359146,0.0020799928,0.5644656,0.32378432,0.6846865,0.122888796,0.8899788,0.32073173,0.22927652,0.19206059,0.6523093,0.5487326,0.28344917,0.9053539,0.1824579,0.8032473,0.6612941,0.38858932,0.9951135,0.0063813403,0.19879508,0.5686374,0.69821244,0.6310895,0.14797159,0.51072764,0.45108482,0.74374926,0.5624344,0.15640213,0.33993185,0.44420674,0.38099658,0.23997723,0.17651366,0.14595789,0.47768867,0.18101792,0.71528196,0.77831876,0.3213533,0.42551002,0.43829197,0.10584792,0.16093779,0.85435015,0.17347516,0.94846594,0.22147967,0.563004,0.7114988,0.82632643,0.23771296,0.21173118,0.09241923,0.682015,0.44126174,0.20367253,0.18859302,0.18273835,0.48110098,0.6079041,0.7237273,0.306874,0.9855829,0.71022916,0.78493065,0.8312051,0.2502492,0.3154562,0.29817635,0.25994375,0.118446425,0.53876376,0.68253094,0.09924465,0.8093488,0.8910314,0.013898499,0.079251744,0.48479113,0.93747604,0.9624969,0.112027526,0.87608,0.6912555,0.7676052,0.21446915,0.9604986,0.91699296]	\N
c5dca646-dc37-413e-8db4-61f9a06b1284	Vũ F	[0.02146394,0.5218651,0.9277645,0.7562386,0.6882776,0.57615966,0.86131644,0.31818205,0.89061874,0.80975586,0.9043094,0.46088916,0.84395576,0.9433454,0.34402764,0.38749588,0.4739268,0.39212346,0.65203893,0.18424839,0.31284142,0.05323924,0.6120945,0.4103354,0.9227798,0.6640753,0.07708987,0.62251884,0.77301353,0.8200894,0.56027997,0.20577955,0.56441265,0.7692822,0.68339014,0.021977633,0.40813828,0.64302224,0.30738473,0.29042652,0.3644443,0.8867074,0.53125364,0.9508376,0.6519987,0.5872686,0.96519846,0.47602373,0.56562835,0.09445904,0.7464392,0.8047984,0.6264406,0.5999352,0.866035,0.6917401,0.5492291,0.5646594,0.30508095,0.68972397,0.5960609,0.69420874,0.95600796,0.80937916,0.71518433,0.8822431,0.3797251,0.81193274,0.71895325,0.9540701,0.5660624,0.9018242,0.38280597,0.3183016,0.249439,0.6000704,0.049336016,0.0069280784,0.8984513,0.59555054,0.3570592,0.14014353,0.82454854,0.4295596,0.7643908,0.5981242,0.30300233,0.64180297,0.6536099,0.3117747,0.007097633,0.1633394,0.6334352,0.032476466,0.13247447,0.052703366,0.8572255,0.9843732,0.4761109,0.0023242391,0.14439851,0.49566066,0.98993,0.6168885,0.08855215,0.7571844,0.6738026,0.93292725,0.51048684,0.68342227,0.20855491,0.20497797,0.41637665,0.8146283,0.85293615,0.39792362,0.37256593,0.8025008,0.82843995,0.7584688,0.87255454,0.097423956,0.058258146,0.2978222,0.86206025,0.63706356,0.4157932,0.44773957,0.65056133,0.7246951,0.081999876,0.81648844,0.94814795,0.47991773,0.0745897,0.6149447,0.1853181,0.68026376,0.9818047,0.3825116,0.9065689,0.34219545,0.05529408,0.3460804,0.28011516,0.3573685,0.54571784,0.38318476,0.6498704,0.7155918,0.8719278,0.12594749,0.82197905,0.20920782,0.064624734,0.32689366,0.65400106,0.35033378,0.63580763,0.63644737,0.41584846,0.5447946,0.081976295,0.63986963,0.47591457,0.67259437,0.036006983,0.43275172,0.06128586,0.4013243,0.38397402,0.39717945,0.016416155,0.34530053,0.45322478,0.91455334,0.16308203,0.16371933,0.2749035,0.6157588,0.5924163,0.085296206,0.71366966,0.78819704,0.699911,0.95104843,0.10154657,0.0007574056,0.131687,0.46307552,0.7996913,0.015397798,0.3145289,0.25073412,0.9034838,0.65343505,0.59176093,0.62910503,0.22444983,0.62685144,0.0634331,0.25748646,0.5209535,0.70520675,0.6507268,0.6926426,0.11674356,0.86033857,0.8870621,0.5851091,0.48584002,0.5997008,0.23582605,0.35630026,0.01645651,0.13184863,0.51927084,0.45520338,0.7545525,0.35126066,0.39499715,0.38537428,0.6151902,0.49244538,0.13398646,0.72008336,0.6782614,0.56099874,0.7515887,0.04846934,0.4145927,0.9250163,0.36041078,0.19594988,0.46198314,0.26811528,0.9849552,0.49784935,0.64884686,0.80145055,0.49066892,0.5640739,0.43154022,0.5413109,0.23739575,0.9529851,0.027499378,0.23788387,0.17193997,0.14331836,0.06485769,0.36189356,0.41312096,0.39851117,0.317826,0.34804702,0.21239622,0.42140874,0.5510299,0.26690477,0.44681558,0.7091846,0.23206018,0.22722504,0.4156634,0.9642936,0.68352294,0.32674265,0.9892462,0.70456177,0.59037083,0.43266028,0.4332699,0.3966918,0.093128175,0.5193429,0.45134768,0.5821781,0.6465609,0.46333882,0.8439652,0.28062743,0.94436353,0.30171338,0.075375564,0.8678513,0.24103323,0.56932116,0.017079836,0.44903058,0.7272086,0.8507411,0.35355896,0.35104364,0.43382972,0.8248185,0.7264198,0.5626347,0.55350983,0.040590316,0.4601896,0.96922827,0.63176936,0.6233163,0.46413168,0.9708076,0.5671339,0.09170957,0.7439625,0.6327131,0.8851439,0.70226055,0.92822087,0.38645044,0.3707397,0.53070104,0.9391172,0.091765374,0.7005304,0.4641155,0.9764615,0.6320061,0.77757484,0.42487255,0.010643182,0.6012471,0.8400245,0.5357842,0.5117766,0.62741625,0.48227397,0.83255154,0.016016565,0.5474966,0.3255327,0.5942683,0.59976363,0.29417402,0.97334456,0.6807411,0.9920882,0.6629895,0.37350714,0.7176263,0.21169974,0.9536291,0.72651833,0.6681209,0.9821549,0.3212834,0.44099835,0.40906003,0.35803276,0.2920078,0.24824269,0.3682646,0.57267225,0.86434394,0.13451903,0.18901432,0.0028332807,0.421442,0.7146536,0.4666613,0.7239089,0.49599272,0.69584626,0.25027207,0.3352151,0.13408598,0.12898685,0.75950927,0.4799781,0.20823015,0.59610254,0.008158626,0.29722744,0.019277975,0.65534633,0.08197121,0.4962222,0.62480503,0.20906346,0.7415457,0.35509807,0.08574679,0.7110405,0.49131665,0.9499289,0.2070285,0.5758055,0.23295304,0.27001792,0.88964885,0.66985655,0.6613423,0.41236037,0.40244326,0.98610693,0.44199273,0.8505986,0.29603067,0.6068717,0.8611965,0.484247,0.9282139,0.15424205,0.046006918,0.7409137,0.3106146,0.94231063,0.20089905,0.9694886,0.47973585,0.08721561,0.10385093,0.0016313393,0.52294457,0.48074117,0.48257208,0.35957402,0.00090586476,0.7320805,0.21019834,0.17654188,0.7120803,0.81525785,0.69101596,0.5706245,0.626283,0.8303736,0.09254915,0.71530986,0.8416618,0.5834534,0.70396906,0.33601564,0.5109965,0.49567366,0.40170702,0.59576744,0.8479475,0.55281913,0.79779106,0.94348806,0.5342774,0.18506119,0.8318717,0.84815365,0.21709296,0.3420914,0.3632636,0.6424858,0.28037342,0.44399688,0.9636604,0.005631927,0.91169727,0.8372166,0.7971787,0.77660716,0.82951516,0.2951705,0.68712413,0.90201676,0.48235676,0.38808662,0.26747337,0.9864942,0.63211477,0.7999236,0.15163265,0.21697035,0.74707377,0.30754462,0.46322486,0.8127188,0.041954704,0.5029329,0.2812903,0.50538915,0.38284907,0.24603242,0.38170004,0.65784585,0.07698395,0.6111334,0.14068128,0.91333526,0.08653285,0.08204729,0.6921134,0.6615791,0.18488048,0.022945799,0.31983092,0.09004742,0.12571993,0.034768533,0.56310076,0.28993982,0.513843,0.6469698,0.24234256,0.3810307,0.64903617,0.033156503,0.9867267,0.1903012,0.47487125,0.8717978,0.6792421]	\N
c94554ed-3ed8-43fb-9cba-716c5f8ecea5	81417210	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]	\N
\.


--
-- TOC entry 3818 (class 0 OID 17007)
-- Dependencies: 226
-- Data for Name: people_count_snapshots; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.people_count_snapshots (snapshot_id, schedule_id, camera_id, people_counter, captured_at, image_path) FROM stdin;
53f1ef23-84a6-4be2-a505-a6fd5ab76b3b	03302383-cb01-4c8c-bd61-9694809df59b	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	26	2025-03-31 13:59:06	person_couter_image/2025/03/31/None__a34800d4-2430-4ab8-8aaa-2568a58d6bc5.jpg
a96719e1-6bee-4852-9a80-87a34f6c8e7e	03302383-cb01-4c8c-bd61-9694809df59b	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	23	2025-03-26 09:47:07	person_couter_image/2025/03/26/None__a34800d4-2430-4ab8-8aaa-2568a58d6bc5.jpg
9a388e2f-63ae-4c85-9d46-6cac57cd26e2	1f7a6921-9fdd-4087-82b5-66aa953ae42d	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	23	2025-04-13 20:19:10	src/person_couter_image/2025/04/13/1f7a6921-9fdd-4087-82b5-66aa953ae42d__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250413_201910.jpg
9253f4d3-eb8b-41b7-a4b3-5eb606e0a743	279be7ea-a84a-49f9-8b33-43419e3eb271	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	23	2025-04-16 21:23:55	src/person_couter_image/2025/04/16/279be7ea-a84a-49f9-8b33-43419e3eb271__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250416_212355.jpg
1aa8d292-c265-4944-8484-b80161ae7c2f	951c502d-abe6-4c5b-9b5c-8f563a0b551f	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	26	2025-04-17 13:58:04	src/person_couter_image/2025/04/17/951c502d-abe6-4c5b-9b5c-8f563a0b551f__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250417_135804.jpg
07dee97d-c1c0-4806-a392-493ac8878970	951c502d-abe6-4c5b-9b5c-8f563a0b551f	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	24	2025-04-17 13:58:06	src/person_couter_image/2025/04/17/951c502d-abe6-4c5b-9b5c-8f563a0b551f__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250417_135806.jpg
68ca7d09-2b23-442d-a47d-a0c7310adbb8	66ccdc85-ae89-414b-bb6f-2f6545444fe2	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	24	2025-04-26 15:02:27	src/person_couter_image/2025/04/26/66ccdc85-ae89-414b-bb6f-2f6545444fe2__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250426_150227.jpg
8993a561-959a-44a3-85dc-bdc2cffa4953	66ccdc85-ae89-414b-bb6f-2f6545444fe2	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	24	2025-04-26 15:02:28	src/person_couter_image/2025/04/26/66ccdc85-ae89-414b-bb6f-2f6545444fe2__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250426_150228.jpg
3dfffc82-bd01-4c64-95e6-3351f68fac06	66ccdc85-ae89-414b-bb6f-2f6545444fe2	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	24	2025-04-26 15:02:30	src/person_couter_image/2025/04/26/66ccdc85-ae89-414b-bb6f-2f6545444fe2__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250426_150230.jpg
f9ca4a61-4aa1-4fd9-8d24-c1b0a5f7ff98	720267c1-dc51-47c1-aa2a-eb904c872a27	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	24	2025-04-27 09:32:31	src/person_couter_image/2025/04/27/720267c1-dc51-47c1-aa2a-eb904c872a27__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250427_093231.jpg
5a11ac4e-b3b3-45bc-a4cf-da1be6c28e85	720267c1-dc51-47c1-aa2a-eb904c872a27	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	23	2025-04-27 09:32:35	src/person_couter_image/2025/04/27/720267c1-dc51-47c1-aa2a-eb904c872a27__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250427_093235.jpg
bf7f285d-ce82-4e0b-82de-4b69979145e9	720267c1-dc51-47c1-aa2a-eb904c872a27	a34800d4-2430-4ab8-8aaa-2568a58d6bc5	24	2025-04-27 09:32:36	src/person_couter_image/2025/04/27/720267c1-dc51-47c1-aa2a-eb904c872a27__a34800d4-2430-4ab8-8aaa-2568a58d6bc5__20250427_093236.jpg
\.


--
-- TOC entry 3816 (class 0 OID 16827)
-- Dependencies: 224
-- Data for Name: schedules; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.schedules (schedule_id, class_id, classroom_id, start_time, end_time, topic, description, created_at) FROM stdin;
d5c52cb1-05c6-4e06-84c3-0748f7ddcf2b	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-05 10:00:00	2025-04-05 12:00:00			2025-04-05 09:08:19.369074
e810b89c-b70e-496a-9338-b954a59f44f8	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-05 10:00:00	2025-04-05 12:00:00			2025-04-05 09:08:33.928114
fffd3747-6a7d-4a76-8618-72f00704a19e	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-05 10:00:00	2025-04-05 12:00:00			2025-04-05 09:10:35.27202
6cac83fd-085d-46b5-9452-0e95832ff51c	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-05 13:00:00	2025-04-05 14:00:00			2025-04-05 09:10:49.903688
71caff65-8347-47fd-83bd-f27adaef632d	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-03-15 08:00:00	2025-03-15 10:00:00		\N	2025-03-14 10:31:05.929207
a7964eef-8265-4c2b-99e4-15f6fb4bb5ce	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-03-15 08:00:00	2025-03-15 10:00:00		\N	2025-03-14 10:31:06.49156
482b53b8-345c-40bd-8871-7c6b020b3a7a	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-03-15 08:00:00	2025-03-15 10:00:00		\N	2025-03-14 10:31:07.114245
cfe45cab-70e7-47a8-a29c-4db782b77fd6	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-03-15 08:00:00	2025-03-15 10:00:00		\N	2025-03-14 10:31:07.615349
87166775-1f1f-416a-ba58-51f8d0933b1f	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-06 11:00:00	2025-04-06 12:00:00			2025-04-06 17:30:19.573794
18fce659-654d-4004-bf0c-71720e0006b8	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-06 11:00:00	2025-04-06 12:00:00	sdfghjkl		2025-04-06 17:39:36.488167
59135269-924e-41b2-9285-af17c3c9fcfe	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-06 20:00:00	2025-04-06 22:00:00			2025-04-06 18:54:09.007508
02516f2d-f313-4464-aa1d-01704c6d6ad3	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-09 16:00:00	2025-04-09 17:00:00			2025-04-09 15:08:07.764225
587da79e-2b3c-46be-9c4c-42e218baff27	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-10 15:00:00	2025-04-10 16:00:00			2025-04-10 13:46:42.212612
1f7a6921-9fdd-4087-82b5-66aa953ae42d	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-13 21:00:00	2025-04-13 23:00:00			2025-04-13 20:17:52.612831
279be7ea-a84a-49f9-8b33-43419e3eb271	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-16 21:00:00	2025-04-16 22:00:00			2025-04-16 20:29:04.1539
e4742bf0-0791-4990-8999-c8f91804dee4	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-16 07:30:00	2025-04-16 11:00:00			2025-04-17 11:25:12.898412
951c502d-abe6-4c5b-9b5c-8f563a0b551f	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-17 15:00:00	2025-04-17 17:00:00	Khóa nội khóa ngoại		2025-04-17 11:25:46.91076
c81c61d9-8b39-4dcf-93f4-1b172267506e	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-23 13:00:00	2025-04-23 15:00:00			2025-04-23 10:44:23.93409
c92ab6b7-ba43-4d02-9b1e-cc462cbeb672	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-23 22:00:00	2025-04-23 23:00:00			2025-04-23 17:03:31.331212
b1b0c3cd-030e-4794-81e6-352e6b0a9f6c	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-24 10:00:00	2025-04-24 11:00:00			2025-04-24 08:54:59.016377
a82c7ca0-fd90-49d6-8d6c-b13a3b4d6537	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-25 16:00:00	2025-04-25 18:00:00			2025-04-25 09:15:14.432079
66ccdc85-ae89-414b-bb6f-2f6545444fe2	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-26 19:00:00	2025-04-26 20:00:00			2025-04-26 14:57:46.501533
720267c1-dc51-47c1-aa2a-eb904c872a27	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	4b26497e-7722-48e6-955c-f369cf16c411	2025-04-27 11:00:00	2025-04-27 12:00:00			2025-04-27 09:30:11.158617
1e9d9b91-b255-4b0a-b7a1-a5e2533eab44	c5ec083b-e4ed-40d3-8f5d-014d839a7617	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-28 00:30:00	2025-03-28 04:30:00	Lập trình hướng đối tượng		2025-03-14 10:31:05.230605
03302383-cb01-4c8c-bd61-9694809df59b	e5f4d4bf-520b-4dbd-8cae-bded98d666c1	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-28 09:00:00	2025-03-28 12:00:00	Lập trình hướng đối tượng		2025-03-13 21:23:53.434617
42512d0c-f415-4061-a50d-51c6acbc852c	0cb59986-50f9-4c0d-86f5-16a5d31f0b4c	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-16 07:30:00	2025-03-16 11:00:00	Cấu trúc dữ liệu giải thuật		2025-03-14 10:31:03.228085
6f7f8106-3ab4-49ee-8b6d-07639cfb5373	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-11 00:00:00	2025-03-11 01:00:00			2025-04-02 10:51:01.820414
c2a75a71-a450-4f3d-888e-8cddf73c44fe	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-10 01:00:00	2025-03-10 02:00:00			2025-04-02 10:51:16.530299
469b6043-4e27-453f-b835-e6ab8805590f	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-04-08 01:00:00	2025-04-08 02:00:00			2025-04-02 10:56:23.18896
36ac642b-79ca-4fea-a88c-6747381fa847	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-28 05:00:00	2025-03-28 06:00:00			2025-04-02 11:18:56.629551
87f09c31-f3c2-41bb-bf52-a60d87191a76	c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-28 06:00:00	2025-03-28 07:04:00			2025-04-02 11:22:16.916777
7e35dd36-809f-4e8b-9fad-fc09c3927754	c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-28 05:00:00	2025-03-28 06:00:00			2025-04-02 11:23:46.200524
2d179479-081c-473f-9ec0-fa16e41b561b	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-27 02:00:00	2025-03-27 03:00:00			2025-04-02 11:24:53.646171
1bf0ad90-7878-4358-ac06-61df790d7861	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-27 02:00:00	2025-03-27 03:00:00			2025-04-02 11:25:06.745496
b069d3a5-2e85-4df8-979d-ef8a00086d31	c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-27 08:00:00	2025-03-27 09:00:00			2025-04-02 11:26:51.12389
4c41c4dc-4b97-46e7-ba80-ed26a7f8309d	c5ec083b-e4ed-40d3-8f5d-014d839a7617	ec997950-900c-4686-a7b5-ccfead53caef	2025-03-12 07:00:00	2025-03-12 08:00:00			2025-04-02 18:28:15.278461
52a28ddd-4058-46d7-8c74-41d6e6a80ba9	25f2ae91-3c2b-470d-823a-6097d9aca4fd	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-14 21:00:00	2025-03-15 20:00:00	Cấu trúc dữ liệu giải thuật		2025-03-14 10:31:08.577061
3a3f6076-171b-4857-8ab0-dba1b91959c4	c7ed9503-a15c-4a60-aea4-6f7ca50c5c60	dcaa7ef5-2946-469b-8e32-55c394dce285	2025-03-15 02:00:00	2025-03-16 01:00:00			2025-04-02 18:38:07.598997
\.


--
-- TOC entry 3820 (class 0 OID 17173)
-- Dependencies: 228
-- Data for Name: semesters; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.semesters (semester_id, semester, academic_year, created_at) FROM stdin;
08c0ba3f-0ddc-4d86-b998-feb3f0d31c2a	2	2024-2025	2025-04-14 08:48:12.561856+07
3bcbf8c5-3056-49ff-a01a-e76853ccb116	1	2024-2025	2025-04-14 08:48:12.561856+07
\.


--
-- TOC entry 3809 (class 0 OID 16729)
-- Dependencies: 217
-- Data for Name: students; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.students (student_id, student_code, face_embedding) FROM stdin;
525fe628-c06f-413e-af2f-b421e5bdcc16	S2	[0.023723295,0.8187597,0.09602709,0.17596123,0.6542114,0.8680672,0.090028286,0.82510984,0.3337158,0.82417274,0.13348252,0.029943105,0.24330042,0.46416363,0.8778458,0.6546771,0.9109128,0.60616326,0.84614587,0.26563805,0.008670498,0.123864114,0.33830673,0.026090458,0.055304844,0.52853614,0.38507164,0.93382293,0.045072377,0.8186193,0.0012352227,0.24084331,0.99921954,0.12030702,0.87900424,0.14677946,0.023952423,0.32586128,0.20764981,0.943371,0.12828675,0.044307183,0.62200135,0.48151746,0.29912588,0.18104681,0.60105276,0.7123004,0.9330858,0.5942178,0.46780583,0.8338256,0.9369324,0.3076205,0.24590535,0.70370173,0.856904,0.18123375,0.6714316,0.1198311,0.88578105,0.03423211,0.9006551,0.88643813,0.8560942,0.83066684,0.893916,0.9050262,0.14518732,0.27723905,0.3976804,0.8761921,0.1220098,0.40132847,0.93452424,0.29472563,0.71396166,0.3980236,0.39455742,0.2595258,0.69825846,0.70371157,0.38640508,0.17400207,0.11564444,0.4776352,0.95332146,0.25779104,0.9554637,0.21099375,0.71468276,0.38579136,0.030630516,0.77657807,0.6647342,0.5643949,0.07468028,0.71260244,0.7770301,0.10132922,0.11632844,0.08045058,0.025330633,0.33598405,0.0467259,0.24667197,0.2243759,0.48000765,0.073324665,0.19538175,0.5819596,0.13826662,0.43194774,0.5516732,0.8070366,0.69456667,0.053323086,0.44879594,0.9014218,0.38452727,0.7731147,0.1111717,0.20976022,0.0057465523,0.57249,0.19819196,0.46353656,0.18133588,0.67431027,0.8966124,0.6626984,0.7584479,0.30077732,0.97794145,0.10591002,0.34072,0.45058227,0.54339224,0.42677557,0.6736917,0.2784762,0.89245385,0.58145314,0.8718486,0.95413566,0.037357833,0.24111292,0.2833029,0.4095188,0.5700548,0.06375422,0.8757998,0.931656,0.36072496,0.77700686,0.5376528,0.566186,0.41724867,0.8611135,0.5821851,0.021929497,0.8973591,0.20402235,0.33172098,0.10423975,0.10005322,0.52132535,0.9201086,0.46168545,0.75351423,0.273434,0.21764998,0.5879991,0.32753903,0.90844953,0.40840557,0.68989396,0.7375716,0.024449086,0.5009919,0.61434686,0.48121768,0.794895,0.6816324,0.42215475,0.54186535,0.6891993,0.23888625,0.86009973,0.32790712,0.09354349,0.29773772,0.60812634,0.83278114,0.24705271,0.4118636,0.2618473,0.46000868,0.15670182,0.24955925,0.32558262,0.27659974,0.23027703,0.6131965,0.18222162,0.22428666,0.2898676,0.41373652,0.81586516,0.22473456,0.23035896,0.78377473,0.24646515,0.29029706,0.6349506,0.13858946,0.9345713,0.45989302,0.26153904,0.76628804,0.37122056,0.14871208,0.51065457,0.19432984,0.9640532,0.25079423,0.5614729,0.6167291,0.35292515,0.654062,0.36046535,0.7263958,0.4899538,0.9317557,0.10441216,0.17252716,0.18870158,0.9813279,0.5489516,0.5951761,0.7676464,0.9507169,0.30240664,0.9909756,0.044691846,0.49923536,0.8508484,0.17248683,0.7179608,0.3159547,0.5608976,0.5258283,0.4283688,0.82637316,0.8574624,0.037764035,0.39769182,0.45789838,0.44910303,0.5541489,0.5187023,0.18973583,0.69994354,0.735911,0.7717562,0.65967494,0.43248546,0.029740868,0.42833742,0.8908433,0.95873874,0.38590494,0.2990636,0.91503614,0.35610533,0.7579171,0.65102434,0.6577259,0.33312196,0.9931399,0.15307452,0.060857456,0.32175842,0.9193534,0.01877086,0.9596349,0.51510924,0.25006938,0.19522753,0.5889201,0.55250925,0.27481332,0.9194098,0.82813436,0.54553366,0.41935995,0.1508008,0.39560005,0.980957,0.5935103,0.75460106,0.27912,0.7668136,0.52328694,0.12391197,0.00485163,0.16177745,0.6695797,0.3225985,0.14198327,0.21130858,0.047786657,0.9126175,0.46600735,0.26600257,0.91275287,0.7918188,0.7946964,0.6250412,0.4506992,0.030555896,0.37476057,0.19451635,0.076813996,0.47078726,0.026341798,0.012105616,0.9799833,0.16107692,0.6399976,0.362358,0.9865414,0.5907074,0.77298963,0.85962653,0.4806512,0.35900855,0.6052055,0.13983068,0.7808052,0.8694407,0.4035962,0.05974964,0.97925395,0.80451065,0.007195357,0.437795,0.112589754,0.18836525,0.7510542,0.20005019,0.37385032,0.5525393,0.7060169,0.73428345,0.32773653,0.7670573,0.8115301,0.09735603,0.6746066,0.81953514,0.77972025,0.26976097,0.97821134,0.6804321,0.28985733,0.91719544,0.5559748,0.06675208,0.8013628,0.2835654,0.62614214,0.5863996,0.80420464,0.048107136,0.11244949,0.38352025,0.39419127,0.30819693,0.86984766,0.53727025,0.32853058,0.4094287,0.26065674,0.6397378,0.65208465,0.7304309,0.000102704034,0.6123812,0.40996134,0.21964224,0.8628938,0.61917555,0.021346817,0.89623535,0.6170481,0.22157533,0.43047565,0.59398013,0.23019728,0.7396388,0.41465017,0.9923288,0.30500215,0.15009429,0.3623304,0.4881484,0.6409399,0.31340244,0.7585187,0.4073907,0.8660617,0.586773,0.9972145,0.7747805,0.41689634,0.6333756,0.07140731,0.492923,0.58248377,0.7994539,0.49226835,0.735516,0.53488576,0.08567548,0.7970812,0.5742225,0.97252095,0.6466236,0.8755586,0.71139336,0.21794198,0.09367913,0.33628768,0.6014084,0.29966328,0.2619858,0.26473087,0.34978577,0.020784175,0.5616476,0.5692729,0.83627033,0.15944709,0.39998165,0.25114,0.99955237,0.7699584,0.14888074,0.6433783,0.29622883,0.25224185,0.7229629,0.94743127,0.45821318,0.9163088,0.18696266,0.74023783,0.2992145,0.07858308,0.691048,0.1161621,0.2837054,0.91949445,0.3721149,0.63826925,0.65240365,0.4573989,0.19024436,0.052587602,0.7055794,0.54697627,0.35006544,0.83550406,0.323563,0.5677529,0.5070167,0.041172575,0.8806658,0.73959774,0.54723275,0.34072027,0.956898,0.1153915,0.04226202,0.7621646,0.9214199,0.69062465,0.5588269,0.22911751,0.92556673,0.8161264,0.114724115,0.78248906,0.83245057,0.6497222,0.05034828,0.6181563,0.7887591,0.22506593,0.06924257,0.83115935,0.13732316,0.2890642,0.258176,0.05513132,0.31660303,0.4853226,0.45952114,0.26308447,0.35880136,0.6375044]
bffc211b-88e3-4a54-9286-01511c608b0e	S13	[0.46723935,0.40407193,0.5262426,0.34910923,0.15804052,0.20110273,0.5831049,0.64708984,0.7846942,0.3459504,0.97915626,0.3969229,0.36125726,0.64810675,0.4108492,0.06619294,0.06358468,0.85735595,0.84581244,0.5348845,0.5532409,0.86164063,0.09160812,0.9783606,0.50928754,0.7928955,0.6983999,0.3097506,0.4492111,0.107358746,0.23136161,0.5582321,0.59474856,0.085517034,0.26025888,0.1445567,0.7699014,0.28989127,0.39201942,0.59289485,0.34157783,0.99746996,0.66454124,0.701385,0.21481273,0.4696655,0.27665746,0.77719283,0.41797373,0.41733903,0.72031003,0.5384288,0.27601126,0.5790787,0.14009707,0.42522174,0.012009923,0.109866954,0.6611007,0.79671144,0.856733,0.4853181,0.049997125,0.11405213,0.02149508,0.3401249,0.8850253,0.15566552,0.09002878,0.9503836,0.51644176,0.21734756,0.7742108,0.685793,0.53759927,0.539354,0.04967448,0.44175202,0.23178,0.75848365,0.88013434,0.33402273,0.78155243,0.7909388,0.8204653,0.270611,0.88152766,0.08017318,0.78990203,0.7894676,0.009048382,0.36443013,0.33065683,0.55279523,0.11936544,0.61766,0.24137068,0.37065175,0.8691403,0.45277572,0.16993174,0.20164905,0.68389654,0.7428227,0.23672667,0.84631735,0.90959275,0.7579462,0.45068586,0.11585661,0.39707524,0.09921625,0.5561833,0.55280995,0.75553125,0.2074867,0.06478163,0.5337889,0.30231664,0.54642886,0.3875486,0.7528652,0.13629586,0.13343428,0.5470128,0.39287153,0.31588015,0.9542878,0.7846295,0.31889728,0.4710296,0.08980725,0.0036505938,0.27195957,0.17343618,0.60676885,0.44140887,0.49090564,0.011870839,0.83438885,0.89168566,0.002762612,0.59514457,0.47524598,0.15192431,0.9555358,0.9743031,0.18871804,0.9346175,0.36898834,0.4738862,0.23394701,0.6895284,0.4848018,0.65677404,0.45304528,0.919136,0.045344375,0.76267827,0.80709326,0.70344186,0.81326586,0.26971155,0.5309395,0.8845331,0.61385566,0.6419753,0.8556291,0.9044877,0.053835828,0.062497202,0.53485245,0.039474886,0.065363094,0.5564903,0.4600637,0.89772034,0.54078543,0.17493342,0.22415432,0.5916116,0.12307879,0.89858633,0.921288,0.5504937,0.3437483,0.80708057,0.71006924,0.24854074,0.78647894,0.09296556,0.84026647,0.27607244,0.92669135,0.71948653,0.73161995,0.54538924,0.81426084,0.31028804,0.06533208,0.2050529,0.5982204,0.5630531,0.79545796,0.98156613,0.8724421,0.50026095,0.6442905,0.20465794,0.76413643,0.48371902,0.5435873,0.066183046,0.17819615,0.9227347,0.03096983,0.8987834,0.48389742,0.4489605,0.6442291,0.900666,0.3080287,0.9511979,0.8056224,0.035178136,0.065213986,0.50364894,0.32486874,0.06288737,0.72263646,0.30794016,0.57202214,0.70253843,0.29301545,0.3217389,0.69336855,0.5288661,0.16127914,0.078845456,0.7616088,0.3514528,0.92209274,0.42339236,0.40718684,0.7496431,0.67194736,0.20913818,0.77725434,0.13793713,0.6337969,0.6933068,0.39836547,0.028627403,0.45358866,0.40030155,0.24118091,0.34576112,0.91590416,0.35122642,0.92116,0.9898469,0.46128616,0.40190363,0.9853526,0.60643226,0.021606114,0.68287,0.0010194535,0.8390408,0.61735696,0.40519807,0.10092634,0.46686652,0.6373628,0.051796794,0.090956256,0.5200065,0.13655792,0.27114943,0.24277046,0.26594695,0.95737886,0.53547305,0.60148823,0.4574095,0.8477879,0.22875223,0.9775845,0.7117621,0.22226356,0.9200823,0.9303062,0.017568992,0.27285236,0.1451036,0.9065447,0.86876774,0.57679397,0.27028963,0.0068199476,0.96809566,0.9913082,0.23661667,0.094008766,0.70180404,0.20896885,0.30243587,0.44207346,0.5264171,0.6161125,0.41834053,0.23816787,0.87324965,0.5397394,0.19511472,0.70707864,0.3466962,0.94647306,0.5506152,0.08542297,0.2354262,0.19853008,0.68288577,0.3383433,0.10129133,0.30432856,0.15185253,0.6671422,0.6600339,0.98835105,0.17164338,0.6268725,0.1660272,0.02101917,0.0997467,0.22971763,0.83212173,0.46147597,0.12753426,0.8079144,0.8415221,0.40022933,0.44272235,0.7012541,0.569073,0.4426744,0.808045,0.030946925,0.25486422,0.8785315,0.8234928,0.5974306,0.57638896,0.8223611,0.948929,0.074898645,0.77055466,0.39193907,0.4204966,0.7363787,0.4366032,0.701147,0.41172814,0.4639123,0.41264397,0.042424146,0.020602278,0.27375516,0.043109793,0.21979567,0.10476483,0.033409327,0.90316975,0.8893775,0.8948776,0.9912685,0.046063673,0.52500856,0.057778113,0.23386148,0.050626136,0.18904312,0.6844498,0.8386495,0.0441017,0.81177026,0.1776412,0.050429825,0.9099676,0.2634105,0.55649143,0.007857399,0.090709485,0.91217124,0.22224903,0.52643865,0.9958955,0.66370064,0.6303704,0.059024867,0.55525446,0.9367428,0.82647616,0.051247716,0.6479644,0.40838578,0.8390244,0.9431257,0.06265604,0.06391258,0.6195409,0.5882239,0.8956385,0.08115289,0.04212647,0.7188487,0.96486527,0.19528429,0.83580095,0.55214673,0.18719114,0.27805173,0.29517734,0.92126185,0.6668801,0.40823802,0.036908243,0.47760206,0.25665912,0.0087511,0.97622806,0.8154282,0.09124421,0.27950385,0.91635555,0.7058692,0.7913632,0.91262335,0.7595918,0.05534981,0.34865287,0.30273882,0.48257047,0.5196408,0.9345958,0.53052646,0.954384,0.2564901,0.4968254,0.5370625,0.6002388,0.663221,0.4055291,0.80640554,0.037261713,0.8279008,0.690254,0.76567745,0.8926323,0.98245066,0.6021897,0.057408318,0.8112295,0.83840615,0.9487754,0.86938274,0.61984015,0.6205565,0.40657696,0.59364474,0.4813512,0.14261451,0.14341937,0.027008234,0.4603098,0.085272804,0.044629626,0.8066051,0.77755255,0.63848495,0.41497207,0.10624439,0.44762775,0.5255422,0.6455109,0.33775136,0.3594344,0.07846465,0.9737784,0.1664459,0.096334256,0.6819763,0.07723907,0.12899642,0.87530935,0.3586777,0.34402052,0.3305471,0.023661846,0.40568534,0.44980434,0.86350065,0.647075,0.042239524,0.9141618,0.10405958,0.76575005,0.4621202,0.30184665,0.12828757,0.24491529,0.761854]
25b90d1d-4e1d-48c8-adcb-a334c517fc2d	S14	[0.80329686,0.42867672,0.8222603,0.98523843,0.6879565,0.7153353,0.1884345,0.30061683,0.32968816,0.7091774,0.72502166,0.1899509,0.3710147,0.49730897,0.41180208,0.21941411,0.38770077,0.44673494,0.65602857,0.7217899,0.45989743,0.9011391,0.58195627,0.13408107,0.8472606,0.07265025,0.25995973,0.22639765,0.6578708,0.26911706,0.5515595,0.6166723,0.5525021,0.9294314,0.38951135,0.7679701,0.89754903,0.88929695,0.98482263,0.07913528,0.7333283,0.30729622,0.25594333,0.0048719957,0.9161384,0.76999426,0.19028556,0.51752126,0.51154363,0.45470586,0.50687134,0.62912667,0.46223804,0.4404781,0.27423444,0.29444465,0.772902,0.54331064,0.5027585,0.70343006,0.41534865,0.9340053,0.92906606,0.56406796,0.18607116,0.47598866,0.5368878,0.20645402,0.9161009,0.5104428,0.5964466,0.60412705,0.3052858,0.40547368,0.6452287,0.32513225,0.02591335,0.58120215,0.027008126,0.8264085,0.56003606,0.4726114,0.48374546,0.25616384,0.12658547,0.9674683,0.9885186,0.53189874,0.7825795,0.6932252,0.55142486,0.9013584,0.86940706,0.26663113,0.8172566,0.45406774,0.1555235,0.5003835,0.01907575,0.19210035,0.040908173,0.7347027,0.87939227,0.10484007,0.21237549,0.861743,0.037530508,0.9431439,0.31453466,0.492343,0.5486086,0.8442155,0.76891637,0.89899796,0.7980362,0.57686126,0.79473823,0.19265714,0.14228505,0.06443367,0.2922143,0.78613794,0.23440354,0.5870857,0.2166338,0.7929226,0.88182974,0.9222071,0.5067342,0.16376176,0.3990896,0.45761657,0.6118388,0.51947075,0.032183778,0.42596313,0.5874542,0.42285582,0.31857118,0.75679934,0.73230433,0.83370554,0.7515061,0.99942076,0.03367714,0.8947213,0.21959443,0.41820323,0.6880742,0.75720024,0.22987106,0.09530969,0.008932004,0.5754095,0.004728895,0.99568164,0.4000061,0.1381219,0.3231435,0.21332863,0.0413294,0.9002236,0.71612155,0.54051167,0.43298668,0.4460691,0.10858592,0.73346066,0.9879708,0.20141533,0.2443016,0.7536761,0.544018,0.7393041,0.8332664,0.032456435,0.95076716,0.35502186,0.5124084,0.99504966,0.21952768,0.51291066,0.9638712,0.7480326,0.06055445,0.09546656,0.36764356,0.5985566,0.26802456,0.66259676,0.77013826,0.2776538,0.82339287,0.66471636,0.097258136,0.78845185,0.14902598,0.2770003,0.35000086,0.84845513,0.26386142,0.34885627,0.02086501,0.2851655,0.4247201,0.5294653,0.5605041,0.08968674,0.12675016,0.9883869,0.5599996,0.9726074,0.55314744,0.6922195,0.70147187,0.14381719,0.6398017,0.0065377937,0.55341244,0.99066544,0.9897666,0.6436516,0.10120901,0.14364758,0.91608256,0.43677625,0.2540908,0.8718321,0.09222836,0.016567633,0.14047334,0.50309384,0.9337917,0.03888846,0.7001067,0.39157853,0.311144,0.73000956,0.66716534,0.50462437,0.9623075,0.9654602,0.60659486,0.6147836,0.9974084,0.99151427,0.58042806,0.24493855,0.112077534,0.502696,0.6053047,0.89011383,0.21124063,0.18249872,0.8635853,0.8250073,0.55763465,0.6343893,0.23631313,0.33218056,0.8377815,0.10203848,0.089759596,0.40375808,0.5127673,0.54691947,0.3685376,0.71285117,0.1991082,0.21818814,0.13375051,0.6405759,0.24875392,0.3929934,0.11191181,0.82207614,0.6485555,0.9045088,0.2860049,0.4561674,0.8532004,0.11997678,0.017309766,0.50073206,0.52680534,0.0556441,0.39383513,0.057065774,0.25211966,0.010163482,0.046560492,0.70824134,0.9117629,0.09254753,0.8988486,0.03952145,0.8192047,0.5115024,0.7770694,0.26251003,0.3371518,0.81037605,0.29338998,0.110802904,0.24159813,0.8400191,0.014083668,0.957935,0.37211445,0.6889282,0.3068046,0.6492206,0.07670717,0.5214044,0.3791626,0.6752963,0.35023823,0.16891378,0.10442723,0.6198773,0.6612539,0.51984537,0.5356185,0.20235029,0.6427057,0.7606212,0.8931213,0.12275033,0.045632817,0.19827761,0.13197006,0.92672867,0.18352868,0.50666124,0.29970902,0.07922171,0.7491581,0.857298,0.24971232,0.5338881,0.10772764,0.16803923,0.99143434,0.631362,0.8496928,0.5214504,0.37166646,0.31295547,0.547106,0.051790584,0.9293459,0.7183486,0.85327524,0.21132423,0.8140344,0.9609289,0.79362965,0.38932362,0.3588675,0.40833002,0.5866757,0.17430145,0.32399213,0.99197745,0.9785421,0.8668395,0.9542081,0.8251915,0.30665573,0.57549316,0.7034346,0.8983889,0.3681272,0.5491739,0.47174993,0.56490684,0.40014076,0.83450896,0.6118792,0.5125701,0.12163841,0.79504126,0.2209614,0.54503715,0.5765716,0.10446668,0.7428221,0.83912885,0.7632729,0.64928615,0.49532425,0.8885923,0.13446562,0.8891898,0.48966008,0.89135134,0.46124846,0.8632003,0.09077258,0.030414946,0.55164737,0.9507313,0.69216734,0.96509874,0.6649822,0.07144716,0.697204,0.9671889,0.6851841,0.997778,0.9692734,0.2484205,0.46324283,0.72532,0.34810552,0.5329666,0.5822315,0.7864308,0.8297014,0.7928207,0.12776619,0.011761212,0.55398124,0.9781671,0.320615,0.45073134,0.57508445,0.03934437,0.68976265,0.68857354,0.56664675,0.116398126,0.8743384,0.058652952,0.24591994,0.25029665,0.31252816,0.78401923,0.105385,0.39283714,0.4049224,0.23285791,0.27252048,0.4203019,0.63502747,0.16658796,0.46398622,0.463205,0.654326,0.060153425,0.6404883,0.67844045,0.97252697,0.09936065,0.8361172,0.44724584,0.53406763,0.616326,0.46129856,0.9128971,0.9317997,0.7151776,0.94862807,0.9708167,0.11868496,0.9444619,0.23528102,0.21732198,0.6086484,0.99487567,0.4923317,0.93561697,0.18539448,0.49972126,0.75847554,0.89683133,0.17949285,0.8925184,0.02315111,0.37906307,0.3177562,0.041013006,0.1307522,0.3695677,0.37533236,0.59855217,0.45506573,0.68707955,0.42388943,0.85072243,0.80941904,0.11914783,0.48642868,0.37236235,0.7992591,0.84005105,0.26606622,0.7217497,0.24387172,0.5643084,0.6502501,0.29277128,0.6868411,0.14540684,0.26678133,0.027630227,0.13401993,0.4606911,0.58674705,0.791146,0.30760115,0.29725197]
821dee23-eaa0-447d-9aa3-f25b6570f98d	S15	[0.5501919,0.31126428,0.6655252,0.6450238,0.879513,0.006941933,0.45853764,0.82238543,0.94321436,0.7161488,0.31292194,0.94474155,0.11417603,0.72097224,0.87219507,0.596467,0.21949136,0.18581831,0.421919,0.029788692,0.091631055,0.65954685,0.09083861,0.010596204,0.06248543,0.37099135,0.68686545,0.3411975,0.06393358,0.7167152,0.7620051,0.07510909,0.9373234,0.12281533,0.6760159,0.62639827,0.46008295,0.6015487,0.5946568,0.29043633,0.2724335,0.8510231,0.14489703,0.16777648,0.05555663,0.43488774,0.14694962,0.6347184,0.7388023,0.7656549,0.26651385,0.25591817,0.8664308,0.40786815,0.7493032,0.83811635,0.29940248,0.8704417,0.94393456,0.4166359,0.42961982,0.30083388,0.9483185,0.6627955,0.7165732,0.3620544,0.017441045,0.21195072,0.8407895,0.9234859,0.30996734,0.5223378,0.36433458,0.49507946,0.759067,0.39602655,0.31167746,0.04414712,0.78456336,0.3564669,0.6894615,0.7386558,0.29245493,0.5059254,0.9480343,0.45871797,0.62595505,0.60440785,0.73968965,0.6089067,0.24388918,0.98562956,0.30898118,0.33946145,0.5393609,0.99178916,0.923614,0.33936775,0.636781,0.03404171,0.07982224,0.4844992,0.47775158,0.17906258,0.77187115,0.68072146,0.6948918,0.90999705,0.62484103,0.27283332,0.034025036,0.15055904,0.08557934,0.06537491,0.9558449,0.2683956,0.24445036,0.037597913,0.07282637,0.7824265,0.4204002,0.12185923,0.16650961,0.40215188,0.45596415,0.60845613,0.86686563,0.39868072,0.42437553,0.28105685,0.81053096,0.6954967,0.6203315,0.38028222,0.058593083,0.016942346,0.31550717,0.4007599,0.048203316,0.79316574,0.11034978,0.9630745,0.967991,0.73671263,0.25501657,0.5752459,0.7576705,0.5455199,0.2662877,0.9233669,0.6942527,0.72841,0.74090314,0.97976667,0.9695078,0.62532437,0.3883222,0.11773596,0.37711915,0.1886653,0.77825695,0.5125989,0.9785751,0.5906698,0.24269827,0.6629318,0.13879146,0.17906679,0.7063131,0.13246834,0.986986,0.09386602,0.36535776,0.8018942,0.6443411,0.53738296,0.70871884,0.8997488,0.19042158,0.53416246,0.12037658,0.3595439,0.75468916,0.2306527,0.16605562,0.7807084,0.3567183,0.7620957,0.44930214,0.2264625,0.96752673,0.6142651,0.33653545,0.19194488,0.9146578,0.3568043,0.9320194,0.80336165,0.18894657,0.6815005,0.39580265,0.17873645,0.7074218,0.9464638,0.5195716,0.9077441,0.76199925,0.22740613,0.22306915,0.28327736,0.73595023,0.5738378,0.12437199,0.074306466,0.19076352,0.7239097,0.2523679,0.14784355,0.4011068,0.8511728,0.54598975,0.431442,0.12881385,0.5183582,0.6080208,0.004150244,0.19481966,0.38977516,0.21985395,0.30303374,0.18102692,0.4643036,0.6800118,0.16107465,0.87919986,0.48907197,0.22708958,0.3014707,0.78792274,0.46423203,0.6185954,0.5718996,0.49085382,0.6251684,0.15121092,0.34114742,0.6216749,0.661203,0.3789468,0.359779,0.51564896,0.4863983,0.77839506,0.8324559,0.709784,0.593976,0.2630604,0.98600143,0.4708003,0.17395414,0.5234423,0.73085356,0.32539764,0.44911483,0.8302427,0.8637508,0.7319471,0.3834643,0.08408981,0.35127094,0.011946516,0.809723,0.18025638,0.7820311,0.94838303,0.12423338,0.08647887,0.8109717,0.5888287,0.4627889,0.90832037,0.4954091,0.036656212,0.20238324,0.2147696,0.9377737,0.5154138,0.151582,0.08164581,0.80017495,0.9390363,0.94292086,0.63048744,0.08857877,0.8627223,0.62375414,0.4751755,0.49816743,0.05765547,0.8787733,0.110517666,0.13244708,0.17119433,0.4568523,0.34414452,0.3466378,0.7383291,0.90420425,0.14874442,0.72710145,0.77443564,0.13434784,0.86478794,0.6402866,0.9411251,0.7784553,0.044586714,0.79492056,0.10720247,0.7830145,0.70749104,0.5690785,0.20056354,0.6121395,0.39268675,0.21528661,0.5563704,0.4509544,0.12776017,0.867475,0.23841256,0.17432311,0.84996,0.41006213,0.46781757,0.5340111,0.36713094,0.66881424,0.96792316,0.95109075,0.7879072,0.80359846,0.5470339,0.5057062,0.5716823,0.46403137,0.79918987,0.25771093,0.5576157,0.16319191,0.22661074,0.12757383,0.30233645,0.054355487,0.19314604,0.4425002,0.33894268,0.15186563,0.94659084,0.7698294,0.03000283,0.6152168,0.13094907,0.9628854,0.38626638,0.42545712,0.75735193,0.37028652,0.160363,0.00054755,0.5017011,0.5338521,0.525246,0.73335785,0.109284945,0.51151764,0.36686623,0.5759301,0.5304131,0.66043967,0.65721947,0.13224024,0.35686982,0.86331975,0.86954504,0.98486745,0.50141096,0.31985357,0.3807207,0.3923265,0.9562931,0.62889063,0.2864209,0.082359195,0.9698006,0.05343327,0.5805276,0.9359035,0.5522882,0.052164428,0.43414816,0.09798223,0.38591456,0.7751573,0.655623,0.50986063,0.21761316,0.8647346,0.42477235,0.22351219,0.6687263,0.08589689,0.11595955,0.90213376,0.32279184,0.014190389,0.9406768,0.016074529,0.2072233,0.7801218,0.3065721,0.19319746,0.7865828,0.76609945,0.65834147,0.08515866,0.7029962,0.7983073,0.44108275,0.654124,0.65327734,0.7594363,0.96554273,0.18197398,0.10514244,0.5481781,0.4500583,0.69668263,0.7500246,0.31817344,0.17743771,0.027115865,0.5588779,0.4583606,0.28503412,0.3205932,0.14062952,0.6161941,0.16980503,0.25122422,0.14235236,0.34097958,0.23043568,0.17647047,0.9584351,0.09164417,0.50913113,0.25871965,0.98396844,0.978885,0.33991084,0.44713366,0.30845827,0.47131762,0.46301448,0.85518336,0.21355872,0.52137685,0.2249718,0.32797325,0.08442547,0.9574073,0.1972907,0.5797463,0.46299013,0.0035105695,0.69680285,0.75262403,0.61250246,0.92507714,0.18232927,0.5154041,0.81745374,0.44922152,0.7073692,0.04310355,0.37846503,0.88262016,0.30534992,0.6013267,0.9861755,0.119187124,0.14831282,0.10387651,0.6987061,0.4017379,0.44390082,0.767529,0.2359624,0.98996973,0.48243687,0.34487516,0.75269955,0.5252702,0.6234829,0.113847435,0.27390018,0.08587627,0.2459272,0.26677984,0.639005,0.16902544]
1dde1982-6705-4d6f-8f47-ddc1cfa83168	S16	[0.22130814,0.2829335,0.06151872,0.45719817,0.7251138,0.7941212,0.5064975,0.008436765,0.9488615,0.8070068,0.54453313,0.30562943,0.70039254,0.3245398,0.69997585,0.015255657,0.10613628,0.14367591,0.106219485,0.07000726,0.5294748,0.3826004,0.24055013,0.8060873,0.14909998,0.9631178,0.030103376,0.44921666,0.12082835,0.41359136,0.883282,0.981327,0.45989972,0.3238226,0.48731428,0.06385867,0.6555641,0.5950126,0.38376907,0.53870946,0.7516635,0.80059475,0.08106264,0.5467829,0.27598935,0.65542936,0.12103222,0.07575689,0.10358161,0.21026169,0.653088,0.3867056,0.3663776,0.875948,0.20881847,0.3335234,0.06821004,0.24308775,0.6497487,0.061783765,0.5106646,0.9744126,0.2637286,0.25790885,0.12065859,0.640078,0.19787537,0.02178317,0.5249123,0.46022543,0.7432359,0.022661915,0.13319401,0.85647964,0.80538267,0.65932906,0.4515776,0.29567957,0.208928,0.95528245,0.7312452,0.55006874,0.05645266,0.053934712,0.8108639,0.031328402,0.91882,0.21913046,0.97149086,0.5714026,0.17487958,0.070058785,0.35654265,0.80427265,0.80097586,0.20009397,0.55053407,0.42920014,0.15389362,0.4078352,0.7604737,0.7221708,0.10067653,0.23429696,0.45562437,0.31637263,0.82100713,0.41094846,0.5706736,0.044321094,0.64036685,0.7309887,0.9364395,0.37825146,0.31300464,0.37331364,0.94107294,0.4715508,0.4050015,0.8222337,0.33052415,0.23984036,0.3365608,0.99982804,0.19711635,0.037072316,0.719283,0.33342692,0.42361757,0.438998,0.3433405,0.3129635,0.7256235,0.009852169,0.61901903,0.41956154,0.18577895,0.7983119,0.60077196,0.48048413,0.259052,0.99093837,0.27515686,0.5481581,0.20639856,0.63812846,0.6191265,0.3943987,0.586084,0.081247866,0.61545676,0.11094019,0.8590055,0.02245374,0.34926727,0.96019167,0.36146823,0.846068,0.24349134,0.28232306,0.71752155,0.84044385,0.043840207,0.86147064,0.89397246,0.74532866,0.001300875,0.7953125,0.17815547,0.15068436,0.4328201,0.47843733,0.7564649,0.38348997,0.94553584,0.39552036,0.90870243,0.43208465,0.37431663,0.65247005,0.20077711,0.6239173,0.8192682,0.72085464,0.26894674,0.053457648,0.9012964,0.4987688,0.4964557,0.16293907,0.5368504,0.5678817,0.9831724,0.28754053,0.54874915,0.64191794,0.8495919,0.81066173,0.69334346,0.5022215,0.24075644,0.925783,0.79578674,0.7678549,0.80514723,0.6317752,0.37584186,0.6406593,0.57446647,0.14879729,0.13588792,0.07242686,0.051128656,0.8507862,0.2800653,0.32904115,0.046885315,0.62337154,0.32219976,0.4201526,0.71705425,0.059128556,0.3568474,0.2734591,0.026620004,0.3430292,0.7159594,0.87210506,0.42764443,0.06855718,0.48352897,0.13256936,0.04626273,0.122453205,0.96032876,0.4850486,0.41879886,0.8796418,0.65960103,0.7911487,0.4817209,0.034498166,0.66001034,0.41169637,0.6325113,0.6352047,0.36593938,0.6830103,0.10574259,0.42860627,0.1534825,0.30807373,0.8008844,0.5336391,0.44331032,0.59899724,0.9191983,0.80142087,0.86939746,0.53989935,0.10381365,0.426515,0.6102513,0.884063,0.9888506,0.85153663,0.5166986,0.1306403,0.39354086,0.8737335,0.3205069,0.62261325,0.56366956,0.6268304,0.33180633,0.05213425,0.1010262,0.4536118,0.8609582,0.5789058,0.43296286,0.0730666,0.3862361,0.0818347,0.31478393,0.52645576,0.43898165,0.87855124,0.46534777,0.38151234,0.4597674,0.9502069,0.804756,0.87239134,0.75335765,0.41595566,0.9649833,0.8354844,0.74510413,0.7029515,0.73123866,0.6633873,0.5808713,0.55724776,0.010802278,0.008388444,0.07907399,0.648041,0.12439754,0.6973958,0.85042566,0.42056268,0.16659038,0.52235883,0.51433736,0.2974039,0.47656924,0.7782555,0.6203801,0.20860623,0.44130751,0.14050704,0.6496341,0.16743825,0.9245851,0.27380708,0.51763684,0.92642903,0.97975385,0.19601896,0.0065108524,0.26586136,0.10115421,0.22608767,0.579762,0.528547,0.26943552,0.48891538,0.87071735,0.033451483,0.5020974,0.55595374,0.69265705,0.1452779,0.12588342,0.6179743,0.09297785,0.9795461,0.99095076,0.10277319,0.7277857,0.092521936,0.9205674,0.59220076,0.19336441,0.4098719,0.25287443,0.65957314,0.045091435,0.14547986,0.8746157,0.939401,0.66501296,0.5640638,0.574674,0.26351225,0.62403685,0.91288745,0.6503859,0.50570405,0.22637944,0.7183832,0.124058984,0.81179905,0.6385543,0.5357359,0.47335538,0.16518313,0.16138682,0.9125991,0.53456384,0.7723325,0.6822985,0.6898672,0.36873484,0.097917005,0.48781532,0.02807325,0.42878962,0.88560575,0.91084707,0.3807332,0.3402162,0.35615754,0.4332131,0.76145065,0.7930261,0.21131435,0.17210926,0.48998278,0.94038,0.022163847,0.9312923,0.35656604,0.9669747,0.3021113,0.88060486,0.43129072,0.27856672,0.98964214,0.65356785,0.8889727,0.9813914,0.28738752,0.8277903,0.66094977,0.41332233,0.42551842,0.6573387,0.32409117,0.7105681,0.0090671955,0.7995722,0.018075328,0.67647,0.9845452,0.55409086,0.6587908,0.6934889,0.025227766,0.04814759,0.10560314,0.31300506,0.032520596,0.1941188,0.9843401,0.6116167,0.3183589,0.65986866,0.06640298,0.94300234,0.9415565,0.494655,0.9080651,0.78931767,0.39269146,0.37254772,0.34358266,0.12617177,0.5338669,0.22863178,0.610581,0.3362535,0.60028696,0.87507284,0.4251334,0.17973162,0.6448011,0.88183635,0.46702254,0.017085072,0.9166322,0.0050616204,0.07032964,0.42971563,0.93452954,0.41362438,0.748642,0.9260881,0.3217686,0.8661377,0.34304017,0.6947746,0.33579266,0.012361581,0.3365601,0.45016372,0.9444313,0.7094893,0.74871004,0.39713955,0.73918,0.06568571,0.8973049,0.45705214,0.36766475,0.92979574,0.3801721,0.68992686,0.82491404,0.3606303,0.3596829,0.8946944,0.7504177,0.4974189,0.3413812,0.6242089,0.78592217,0.31852084,0.86485714,0.5210014,0.520007,0.49532223,0.005246465,0.031540252,0.051364858,0.47570595,0.488299,0.06973565,0.2786242,0.31771743,0.08855911]
bfea3fec-742d-4cab-abbc-a0fd90827cb1	S17	[0.3123207,0.9525555,0.6458031,0.67908263,0.08824453,0.87455827,0.82549363,0.77353215,0.19910812,0.6296506,0.066437185,0.13737585,0.10617101,0.31309667,0.612189,0.79590744,0.9154018,0.70155877,0.48643902,0.64872926,0.43475765,0.78802174,0.05753914,0.49481243,0.18290156,0.57611865,0.7580603,0.117189415,0.00961332,0.9906944,0.36062524,0.057212666,0.11208593,0.69147253,0.7276228,0.21671669,0.80174947,0.3653454,0.83521575,0.9718313,0.23365,0.058604166,0.09195928,0.7221703,0.78529805,0.9080192,0.8392342,0.84089583,0.0521774,0.33681014,0.68317825,0.1528417,0.12808461,0.91974294,0.7501739,0.98273546,0.8073192,0.37457386,0.9775767,0.005767668,0.44106945,0.029376948,0.03662228,0.08807776,0.6741834,0.36533734,0.8706939,0.03223847,0.5868343,0.33702114,0.8870455,0.09355679,0.10434421,0.6222192,0.9454929,0.5865096,0.44484496,0.9660951,0.1915714,0.72660697,0.40414643,0.96865296,0.3526212,0.5419581,0.19889559,0.9927411,0.96237034,0.03092744,0.8240893,0.5150249,0.6901048,0.7717249,0.32867834,0.438396,0.9887045,0.5050953,0.8136643,0.2967227,0.30002427,0.22360973,0.34715974,0.8055953,0.7725038,0.844164,0.65132666,0.760883,0.06032286,0.0056755063,0.23405518,0.57710874,0.26464552,0.9030846,0.28183368,0.5770711,0.27578184,0.68288434,0.7617813,0.88161606,0.10537894,0.12718506,0.5714898,0.15925659,0.8148776,0.7004655,0.7942274,0.97709936,0.60285723,0.3484485,0.20886026,0.6352567,0.5806841,0.5016891,0.6468954,0.9807991,0.65339595,0.18398514,0.6255497,0.513057,0.5671563,0.3233474,0.3971254,0.7409899,0.86208916,0.09204789,0.57286286,0.5260591,0.25503093,0.44958022,0.731199,0.7473733,0.2958245,0.11880258,0.47891536,0.67005306,0.7709146,0.027095871,0.22804615,0.6626086,0.2707573,0.54962945,0.7382576,0.8474478,0.5318975,0.22648077,0.14177641,0.4539,0.23151445,0.7431064,0.7007846,0.2946625,0.052793454,0.48153818,0.3675534,0.14342438,0.3263328,0.31053302,0.7196769,0.75105214,0.97666657,0.26591632,0.5792037,0.51948273,0.21407856,0.014959136,0.37460312,0.92795527,0.8760348,0.9607312,0.24765919,0.9528261,0.5026866,0.79333466,0.5295248,0.6994146,0.43122625,0.27112427,0.6578572,0.17328319,0.393129,0.6768878,0.6602036,0.92472243,0.73073393,0.2295436,0.5760897,0.53513193,0.31883422,0.7975872,0.6399342,0.929076,0.39626318,0.42141673,0.96139514,0.14788173,0.7008098,0.45922416,0.5933842,0.32473075,0.96281004,0.7092926,0.49477872,0.6255554,0.54681516,0.60399646,0.5025234,0.56767994,0.87024283,0.17240117,0.8925173,0.26493114,0.795081,0.076135024,0.5419747,0.42248523,0.54894507,0.52909636,0.67325985,0.50496143,0.82574975,0.39903677,0.46437964,0.661342,0.68028355,0.67485595,0.7954279,0.13139103,0.8194575,0.75874895,0.9791838,0.100721836,0.26081622,0.5517827,0.87444097,0.16684645,0.051777557,0.9797315,0.745121,0.06950473,0.59400284,0.44284314,0.7930042,0.11116021,0.9047948,0.35606167,0.29741588,0.92301875,0.6865179,0.046916347,0.79281265,0.9173588,0.6949139,0.65900636,0.047598843,0.23987377,0.6301995,0.88749975,0.6544114,0.8868902,0.6521537,0.2564487,0.64713115,0.0063781,0.09192011,0.44142145,0.45079783,0.39355105,0.1409152,0.5970491,0.9495945,0.18812057,0.637218,0.9577216,0.8524239,0.71500105,0.545967,0.2316618,0.75468916,0.8293787,0.50208604,0.16567212,0.459468,0.24804433,0.37371227,0.5589942,0.8261659,0.6730616,0.96470433,0.97085917,0.0041817375,0.61085606,0.23581891,0.11688447,0.13499695,0.09909545,0.32393944,0.3005438,0.36343864,0.052781988,0.7219167,0.91341513,0.20678218,0.80919725,0.63643074,0.97712034,0.6915626,0.5652007,0.0011390563,0.4876166,0.52956307,0.656937,0.48614395,0.65947515,0.027612519,0.113635786,0.98457366,0.35325637,0.26013154,0.30583978,0.24255264,0.6045464,0.13698605,0.64335454,0.31177598,0.31711182,0.047707327,0.29549614,0.0013688932,0.89790654,0.03969713,0.9001318,0.42857093,0.19908638,0.9399921,0.8788844,0.85317785,0.15763858,0.15371956,0.25014302,0.7882889,0.50725955,0.7936116,0.0698488,0.61985654,0.8189838,0.7804373,0.62941873,0.5222067,0.4750686,0.18771254,0.6978947,0.8627649,0.7666238,0.10115733,0.47298983,0.6906206,0.06462839,0.8940209,0.6201058,0.5141389,0.28617367,0.91089344,0.3521873,0.83726966,0.90194464,0.18851683,0.3936458,0.436148,0.5008392,0.14414291,0.22184525,0.2021907,0.5494878,0.6461958,0.7060595,0.26366016,0.94652396,0.11013399,0.018897811,0.27904475,0.088598184,0.5185053,0.44444287,0.41813728,0.14944746,0.5741347,0.46904728,0.528339,0.22855699,0.64929056,0.34909913,0.95533234,0.120964445,0.25149518,0.6372551,0.53055066,0.07009508,0.9931945,0.5929908,0.026617214,0.7870369,0.79626095,0.48221466,0.40100762,0.4062163,0.5664124,0.25432464,0.86048436,0.89902633,0.041127495,0.5318854,0.95728105,0.76448727,0.5586219,0.675793,0.77949166,0.26146275,0.5571237,0.5986109,0.43168646,0.455589,0.9314414,0.31548184,0.89545375,0.7396278,0.5353027,0.40849373,0.3740533,0.81423676,0.94179255,0.65057737,0.43381804,0.77561927,0.41525474,0.2904272,0.7670652,0.98101544,0.7057858,0.511043,0.8017541,0.89315385,0.524787,0.99373645,0.26533434,0.3470447,0.64934254,0.29181418,0.61645645,0.23221914,0.03242171,0.35997736,0.67155004,0.72621536,0.04593909,0.0037014338,0.37883636,0.30945846,0.4125973,0.9324571,0.78801596,0.10922177,0.34406662,0.03704034,0.73683363,0.40191248,0.5713098,0.1272123,0.12210808,0.7397229,0.5927324,0.33181015,0.8069879,0.5765488,0.17960247,0.88934,0.66117495,0.49289837,0.5984425,0.6829372,0.66934174,0.34827808,0.5402794,0.35045448,0.4432689,0.6247581,0.5005807,0.7367163,0.8330259,0.55472016,0.15847382,0.34637395,0.44123355,0.15005626]
b6c748f3-d21c-48e4-a975-f604ebd9240a	S19	[0.94233525,0.2369349,0.12934528,0.38748774,0.6440691,0.6293862,0.9442152,0.7089574,0.048241448,0.30242494,0.03859693,0.7477425,0.034258362,0.9773336,0.6914564,0.73810107,0.8584727,0.40210795,0.7095182,0.98467094,0.0006304928,0.47123638,0.037162583,0.19153972,0.63254035,0.16048588,0.85375595,0.49052414,0.29483572,0.82429826,0.28853056,0.987028,0.02908132,0.38191766,0.61751616,0.54013276,0.84188306,0.9770901,0.090620495,0.91816634,0.9545989,0.29138723,0.1870005,0.024980329,0.39870256,0.8416306,0.19671242,0.88308746,0.36918235,0.63595265,0.8165274,0.11175889,0.24046978,0.6440475,0.34737587,0.00268091,0.15082814,0.43054414,0.7087413,0.9274536,0.8454213,0.87065774,0.5625043,0.33726957,0.6129007,0.38206342,0.6796504,0.5867819,0.9792404,0.23618543,0.081708826,0.460017,0.93128896,0.757501,0.046515398,0.90050566,0.59130794,0.56360835,0.25246885,0.74331826,0.16349782,0.42092863,0.9018818,0.26613975,0.27521142,0.32998797,0.6207747,0.07447994,0.32630566,0.87960553,0.7864965,0.44013265,0.09019001,0.47251603,0.41280478,0.88428897,0.3105781,0.638239,0.6478235,0.6512301,0.37312397,0.07457235,0.51147604,0.7739062,0.14582285,0.7352625,0.92482555,0.037875164,0.69958496,0.46535346,0.53250605,0.95697075,0.561068,0.814285,0.7801708,0.85240084,0.9844993,0.79651505,0.93930054,0.24983853,0.78533006,0.7519529,0.41906,0.31682312,0.55684453,0.06966278,0.6792142,0.14372432,0.38870406,0.9999668,0.9018944,0.25236356,0.106421396,0.2701908,0.30687946,0.533951,0.6370856,0.9465293,0.7166962,0.232676,0.6122414,0.093434185,0.41299132,0.111289695,0.3729002,0.47981858,0.242407,0.4199737,0.94868094,0.8181413,0.032163266,0.62176144,0.7693587,0.78270876,0.7126903,0.36521494,0.5768361,0.36305138,0.28603056,0.5451738,0.6130817,0.06949332,0.70137316,0.38193992,0.216947,0.72206104,0.0075589935,0.7018849,0.50639915,0.16801193,0.22049889,0.4076647,0.07328497,0.32861656,0.35950184,0.6516011,0.3036677,0.7045549,0.44427663,0.83029276,0.88560593,0.39144728,0.7763614,0.31436878,0.74295306,0.09981839,0.6337564,0.10511472,0.9940892,0.8896898,0.24746823,0.2559248,0.89578676,0.7222387,0.9201751,0.47208756,0.06406044,0.30088687,0.15419258,0.7648826,0.30204415,0.7468109,0.24276352,0.31943333,0.12941176,0.5010155,0.12023975,0.26978728,0.87146515,0.56320673,0.80039257,0.8099033,0.047546327,0.4260828,0.7418341,0.09609328,0.7980797,0.968186,0.8685755,0.9658985,0.52768403,0.37370348,0.20696619,0.86015826,0.9360629,0.6437982,0.874463,0.5541511,0.74551094,0.01194577,0.20605695,0.921048,0.933426,0.9065188,0.5204469,0.3775109,0.64885646,0.2140333,0.29442838,0.21317278,0.38572502,0.4229003,0.47807747,0.83377635,0.5810745,0.2544585,0.652918,0.96730566,0.15674688,0.45995158,0.08531229,0.2672919,0.25970915,0.63556033,0.3618077,0.75750524,0.53200454,0.97563696,0.8255609,0.4747196,0.13335964,0.5417504,0.29058748,0.3493559,0.98137546,0.9710114,0.9980272,0.8285084,0.13544248,0.98644614,0.073568374,0.26016882,0.66041446,0.26716316,0.13619988,0.17250898,0.47014427,0.06449776,0.8070596,0.5714522,0.89063627,0.18261594,0.24309535,0.9104388,0.08447139,0.5897556,0.64334637,0.8017326,0.6829281,0.92214566,0.98832893,0.04783904,0.07003484,0.4180894,0.96339417,0.24691342,0.8097586,0.27501398,0.088139884,0.30442137,0.90909153,0.8298443,0.257058,0.5339076,0.030051848,0.526981,0.18149573,0.11308402,0.07377737,0.049197912,0.14298841,0.81732494,0.5307034,0.2816155,0.4833995,0.65455663,0.041793793,0.95458925,0.4110141,0.46960324,0.45904258,0.40917557,0.7250276,0.034017276,0.9989736,0.42364147,0.12022945,0.08130796,0.16282012,0.24655405,0.096654385,0.15879668,0.2460405,0.5324991,0.90374744,0.9053359,0.25466633,0.49980628,0.77712137,0.9146856,0.29062143,0.29898116,0.4510792,0.56784,0.71413666,0.8813496,0.08160566,0.6192217,0.35262352,0.38056797,0.64122295,0.07176763,0.38162637,0.43121845,0.051299892,0.27935943,0.3188323,0.5694148,0.6387007,0.553689,0.39111215,0.838048,0.10966382,0.5893733,0.45101604,0.85981035,0.37379304,0.9728561,0.90985185,0.36632013,0.07007256,0.4175247,0.5837505,0.1950679,0.24634102,0.64555424,0.15322493,0.83521783,0.5343184,0.62744737,0.52437353,0.49935868,0.25732023,0.33825657,0.37940282,0.71058655,0.76060534,0.24740218,0.88700205,0.32993197,0.499727,0.77966315,0.15895545,0.8167253,0.08466084,0.18634549,0.26385346,0.22471404,0.96609867,0.5070366,0.8020086,0.44557062,0.1755458,0.15445168,0.65470314,0.55740225,0.9451187,0.62062055,0.22385383,0.3632143,0.98439217,0.3231248,0.92290246,0.11513589,0.8774893,0.79583013,0.2738776,0.38910654,0.6399834,0.4971927,0.5259492,0.9807247,0.825584,0.14467369,0.7705318,0.7826374,0.21234977,0.4724642,0.7699091,0.013357329,0.7359036,0.3611062,0.7647397,0.018362138,0.09976955,0.720171,0.8065584,0.2946802,0.9265898,0.15956537,0.04038696,0.106107555,0.7620668,0.3245452,0.666235,0.075040385,0.1188705,0.32590213,0.9645802,0.45696127,0.69294375,0.3858913,0.7206896,0.8184593,0.9347226,0.10602128,0.6701105,0.8586348,0.5549634,0.077340364,0.85594136,0.97441274,0.72293264,0.46404397,0.67715496,0.5040251,0.35500863,0.12730609,0.85206324,0.2383984,0.7708155,0.87544155,0.8820662,0.8665327,0.23808081,0.07587007,0.2422003,0.59475034,0.7966061,0.7268051,0.72779727,0.7278932,0.1846384,0.52099,0.15939896,0.6127006,0.14436884,0.1908702,0.95906085,0.7308737,0.63268,0.31650484,0.86699635,0.5912927,0.7614915,0.4324919,0.6283196,0.43888825,0.43163958,0.9292641,0.866524,0.8641687,0.26605776,0.9317202,0.037552193,0.5872984,0.21550348,0.4739344,0.07893606,0.09966905,0.2777377,0.92282194]
29a519d0-5c29-4496-9687-1b7942dbd7ff	S20	[0.30438763,0.6952436,0.77289873,0.3058171,0.21490343,0.37827727,0.4564369,0.50917196,0.9827202,0.40346393,0.067488894,0.5479473,0.88354975,0.5117687,0.7394409,0.5328246,0.45887175,0.045822743,0.037435,0.086061664,0.66819423,0.30910835,0.88385564,0.3809327,0.25361124,0.7854738,0.39310303,0.50594056,0.78541535,0.54955524,0.49382824,0.56548667,0.41533673,0.36316767,0.8880996,0.6980416,0.07261735,0.6519787,0.07430561,0.057304163,0.40547726,0.5983216,0.840482,0.18483837,0.71322745,0.80835444,0.76753753,0.57642376,0.46491352,0.44379383,0.8684425,0.5128805,0.18544666,0.15500727,0.74757516,0.25633425,0.78794134,0.22302787,0.2766049,0.21356054,0.5230003,0.7462355,0.9253459,0.55862695,0.4030826,0.16335885,0.6114049,0.7567773,0.76403624,0.36761227,0.03396215,0.8531306,0.33775887,0.8056637,0.72338796,0.54383445,0.82145363,0.81335026,0.92872506,0.8616742,0.5250751,0.13660707,0.05426476,0.06420203,0.46284208,0.80191815,0.06598371,0.7216901,0.55194896,0.33083946,0.42304438,0.8421538,0.7887727,0.31704813,0.9597264,0.18830492,0.6208843,0.2001471,0.45418727,0.16882174,0.6363864,0.22216414,0.29760155,0.30888802,0.3661756,0.43058592,0.6545632,0.093051925,0.41516337,0.5014734,0.067623265,0.29585078,0.75507057,0.22123007,0.353705,0.17437963,0.6373227,0.7296981,0.3847594,0.23721607,0.8588807,0.2697762,0.8945145,0.27212372,0.63525134,0.3927797,0.7830856,0.8405324,0.88373554,0.40362757,0.52009255,0.17891376,0.67317814,0.0962054,0.41540396,0.25693417,0.4048069,0.8415636,0.9123874,0.24537982,0.99671036,0.93656605,0.11227216,0.35544506,0.557894,0.7916734,0.5928745,0.40657687,0.8706068,0.5859731,0.18814512,0.012533698,0.8136841,0.62272704,0.9110405,0.07049102,0.78078854,0.3613427,0.8129384,0.98064685,0.12950695,0.20623918,0.90973294,0.37582347,0.77606636,0.30841213,0.8805687,0.24911912,0.5379966,0.7279882,0.8568484,0.02241423,0.7562103,0.65849316,0.5532751,0.23608337,0.018655226,0.9592443,0.27749053,0.9548284,0.3506425,0.42868623,0.21939193,0.19195043,0.7045073,0.596565,0.5067815,0.007989967,0.08329669,0.009481924,0.8418515,0.32679784,0.6936747,0.8624493,0.60603565,0.546376,0.8882949,0.96498924,0.6772176,0.15160817,0.63127726,0.17487489,0.7863557,0.42907652,0.08328578,0.0016436287,0.40821213,0.8204045,0.14129035,0.06832345,0.3264363,0.38416174,0.3741995,0.08329805,0.44074202,0.8260758,0.8385041,0.6229053,0.23987602,0.54208016,0.08106876,0.24067283,0.7240035,0.5783344,0.4569815,0.9978939,0.5393716,0.06760307,0.9315923,0.14481124,0.1825731,0.7596679,0.09539472,0.39349508,0.57614815,0.38059476,0.11743725,0.3927042,0.9092574,0.82947385,0.82134265,0.7907993,0.48969594,0.5968975,0.03553276,0.44605812,0.2633868,0.53790116,0.34580657,0.22709633,0.5534731,0.7212798,0.87588936,0.22553276,0.24687575,0.013620282,0.26152334,0.7484208,0.35843435,0.566774,0.39643127,0.89319366,0.29482237,0.69452834,0.12935866,0.62416565,0.7160364,0.98152363,0.33359742,0.8462587,0.33255076,0.46643966,0.31868804,0.3744196,0.3436499,0.9733108,0.4812305,0.8097165,0.89095646,0.44328827,0.60980994,0.38886264,0.49532947,0.23798895,0.7690227,0.084018804,0.16471355,0.579609,0.97808963,0.32252166,0.2889431,0.064596176,0.6271777,0.010456043,0.40537998,0.32428437,0.93951046,0.830418,0.47215912,0.143811,0.35456967,0.524758,0.30692512,0.6915228,0.6634318,0.34677395,0.96866494,0.8225644,0.91804963,0.40453148,0.83373034,0.13497853,0.93908787,0.17100173,0.71790016,0.9367305,0.08707699,0.8066046,0.51636416,0.49579537,0.25617757,0.25957036,0.3976315,0.47494614,0.723456,0.42154974,0.7899355,0.39651996,0.9387292,0.1688095,0.6461934,0.046301395,0.777416,0.90028846,0.35925493,0.49544367,0.4264736,0.1406087,0.23224732,0.8996463,0.66355443,0.72921616,0.4854483,0.8929521,0.7454397,0.20240901,0.10945811,0.0655397,0.91824096,0.63090694,0.8737476,0.28415072,0.7929434,0.9605482,0.30388346,0.07122911,0.46197855,0.41781494,0.02365755,0.6495747,0.29469943,0.19083337,0.5379714,0.11475885,0.53358483,0.1123385,0.3304277,0.8216819,0.70738965,0.026228227,0.5683727,0.9560002,0.9308819,0.62804306,0.4369422,0.376356,0.3475475,0.58136094,0.47262305,0.65882665,0.60635513,0.014239921,0.5826486,0.71445525,0.83252734,0.5228265,0.9426823,0.9729504,0.9664863,0.36224988,0.17084041,0.37490657,0.104597926,0.2025983,0.05475036,0.18839146,0.09122933,0.50292325,0.31708178,0.6364486,0.20317063,0.9579137,0.89372295,0.97765535,0.82025486,0.6512337,0.41400817,0.38604158,0.4113682,0.9228392,0.6050908,0.47157392,0.57970726,0.62236387,0.42846632,0.30128276,0.0840595,0.091475226,0.67986035,0.006008792,0.8002685,0.63983303,0.448621,0.68495476,0.9389891,0.66890585,0.6131908,0.90977955,0.19083923,0.9177835,0.90767825,0.8715217,0.7619179,0.32279027,0.59180677,0.4854344,0.7831573,0.10497997,0.20903985,0.00987606,0.93014026,0.78624904,0.6371768,0.024670275,0.15222554,0.6092838,0.89342296,0.7803174,0.4374581,0.18284984,0.2237581,0.692427,0.13521414,0.5945362,0.28880885,0.3325906,0.37184206,0.16352567,0.4899088,0.25141686,0.9435864,0.24931847,0.2320876,0.7464116,0.9593879,0.32802472,0.08386786,0.5551903,0.6637517,0.24684416,0.5701509,0.25194252,0.7545268,0.52865446,0.83673835,0.8797881,0.5878621,0.50708985,0.9764328,0.6996086,0.30340782,0.8374153,0.27073205,0.58431864,0.36021966,0.61776143,0.23659973,0.61061406,0.4899206,0.72800684,0.008650894,0.80985534,0.54325473,0.1443572,0.6451764,0.722267,0.35237482,0.2459945,0.7630417,0.20899306,0.94683224,0.06491963,0.7465815,0.74750704,0.70603627,0.8237201,0.31622243,0.0039036735,0.36421168,0.83365756,0.8057871,0.7874316]
9977e3f2-a43f-4e5a-9115-6ba44bc54cec	S21	[0.7118018,0.41553557,0.022944223,0.85107946,0.8781435,0.21297765,0.443437,0.088282496,0.17028336,0.19301663,0.17479312,0.62047637,0.49559316,0.590929,0.6154067,0.7849315,0.95107394,0.0775253,0.5820191,0.6504072,0.61194944,0.3722656,0.22844663,0.44100285,0.059561584,0.91180277,0.48103455,0.28174478,0.6937409,0.44366506,0.25375676,0.54080683,0.5194779,0.65750295,0.44790682,0.52901715,0.50030327,0.036831677,0.8049717,2.5667829e-05,0.3867447,0.7076505,0.11044565,0.18688782,0.94562685,0.5400604,0.6311035,0.058424614,0.37559098,0.25037342,0.08304973,0.3450965,0.7828343,0.34789687,0.20152389,0.035177965,0.9388459,0.14876048,0.09736731,0.63348836,0.96518797,0.5051173,0.009426816,0.28280127,0.9300359,0.50469697,0.71527827,0.8378052,0.5000172,0.41649798,0.8051267,0.76193154,0.8983926,0.59582216,0.15607895,0.6922451,0.8198235,0.9923616,0.6953059,0.76194906,0.7296044,0.42884412,0.22302765,0.7464791,0.9065241,0.568512,0.69273865,0.24430318,0.8399683,0.8057341,0.22390355,0.763823,0.4019706,0.32615048,0.5647128,0.80820405,0.34175378,0.65247554,0.28219625,0.6266098,0.9082472,0.8819113,0.51325846,0.5304279,0.3971697,0.12551714,0.26249564,0.66916895,0.4693016,0.86100113,0.022357566,0.8566306,0.8123221,0.31755665,0.41912204,0.020530734,0.9039504,0.6085015,0.49620324,0.5320087,0.034782365,0.9958638,0.3281777,0.098808214,0.60348463,0.2184543,0.28642622,0.91717947,0.85118955,0.14434586,0.13797607,0.9206629,0.010150001,0.42916396,0.34529603,0.40163246,0.7768298,0.18224533,0.78447604,0.5258197,0.1564092,0.6621396,0.48592907,0.36598474,0.8929851,0.68418276,0.080775835,0.74037397,0.7186753,0.784228,0.21773401,0.72232527,0.15339161,0.63144916,0.038782142,0.21791825,0.05860419,0.72586167,0.8008207,0.61327314,0.2634979,0.6849815,0.09793939,0.7697878,0.8193027,0.49404448,0.63037413,0.79979074,0.07124743,0.59818137,0.21720707,0.8774167,0.6669371,0.7982635,0.8495858,0.6226545,0.9611536,0.3541951,0.90717393,0.41712695,0.5973148,0.9470261,0.6004519,0.4464515,0.97611547,0.49792746,0.44859093,0.1487607,0.3756151,0.061224204,0.6895946,0.36663082,0.4353183,0.98670995,0.9677861,0.51664734,0.11924519,0.86425775,0.8389519,0.9792254,0.7392895,0.84822065,0.57570136,0.900274,0.2877349,0.033848293,0.90464926,0.46737814,0.35419825,0.02593909,0.31626242,0.40352714,0.86407506,0.7880576,0.59276795,0.33714297,0.9511169,0.84051365,0.7351435,0.0060994537,0.7258795,0.12739755,0.58987325,0.41175428,0.6264065,0.4550768,0.3984433,0.54604185,0.2906556,0.25145498,0.6175742,0.24194308,0.6357921,0.573459,0.22274661,0.28619954,0.42354974,0.6500183,0.57547164,0.39492342,0.3621334,0.7296091,0.2798344,0.2814896,0.24542612,0.5037901,0.515403,0.5423228,0.4468648,0.48076537,0.868573,0.89657813,0.67574584,0.44185823,0.071945004,0.4063657,0.8579483,0.9090002,0.09869453,0.49576637,0.947516,0.05926478,0.77383167,0.88782936,0.8751428,0.85549396,0.079095095,0.109290384,0.17354892,0.10602642,0.23470324,0.32286885,0.34245285,0.16688327,0.3236289,0.37331688,0.15868726,0.27221924,0.80752,0.89275426,0.7796825,0.9649733,0.29262444,0.5439848,0.9625555,0.57431,0.30497214,0.45590112,0.6202611,0.7469706,0.5288757,0.315674,0.02492094,0.80873233,0.8222741,0.38515085,0.30755442,0.11384415,0.19457425,0.6798171,0.6158545,0.6292932,0.7220827,0.5653383,0.8602609,0.26957214,0.6259555,0.73712593,0.3855027,0.8047542,0.013062631,0.70525885,0.11036442,0.5837726,0.7060058,0.84172547,0.12846191,0.36580372,0.99885887,0.090577416,0.5833668,0.62037736,0.49167508,0.84401375,0.42628583,0.7093236,0.2538637,0.5226997,0.6449748,0.6771781,0.6181336,0.2032844,0.87843597,0.23330107,0.9253711,0.4668597,0.793594,0.15289599,0.92266846,0.8102568,0.9179526,0.75522155,0.36729002,0.64643526,0.6732585,0.23331402,0.10108393,0.082173355,0.5489654,0.54723567,0.8192875,0.6311914,0.99180317,0.12153745,0.7920108,0.5124981,0.14053528,0.46631616,0.28965396,0.44879493,0.2600483,0.35788018,0.8626716,0.7792108,0.3794046,0.738492,0.6549069,0.9829938,0.23232546,0.42186326,0.72148234,0.79717994,0.8558176,0.6052522,0.51269656,0.99811894,0.418081,0.54090697,0.37819073,0.18287975,0.26205617,0.7628981,0.37875304,0.2823172,0.076762676,0.85666764,0.2259495,0.8577383,0.15239155,0.13612159,0.1466537,0.091658086,0.67773765,0.5009772,0.3823455,0.7711202,0.48285568,0.4720943,0.75536996,0.8316264,0.0004257564,0.6809519,0.71565235,0.4960852,0.34598875,0.44236743,0.6571089,0.10453054,0.20546778,0.9525087,0.32654285,0.7652988,0.4198136,0.988666,0.11429648,0.080742225,0.77865577,0.11175213,0.55499697,0.5596035,0.8762077,0.9960492,0.12633994,0.44307187,0.6559079,0.35860977,0.60469556,0.9724906,0.0051516495,0.42292786,0.46961412,0.32837075,0.96170545,0.57149225,0.2456391,0.54612565,0.89603776,0.65391684,0.9710042,0.6564261,0.048238292,0.6204977,0.7329773,0.90942764,0.6984788,0.18648732,0.31694487,0.4488567,0.43764472,0.114743695,0.99913895,0.21995495,0.39932728,0.49379584,0.92345726,0.46823347,0.41930208,0.41020685,0.7381642,0.028585054,0.8822444,0.95665634,0.1903433,0.39051777,0.7337252,0.8855136,0.9948771,0.81068635,0.8078511,0.107220225,0.7412241,0.68458575,0.70511097,0.8030166,0.48380226,0.59934026,0.5188365,0.75592023,0.7865459,0.7356941,0.93812263,0.64356923,0.527661,0.40480298,0.32043177,0.278807,0.67757225,0.49713674,0.51421505,0.42496842,0.9970532,0.35142326,0.42659757,0.9209442,0.14088,0.8821244,0.5150125,0.54155695,0.1850273,0.34242365,0.43016544,0.005974471,0.7529552,0.72169673,0.6828037,0.5643907,0.42023185,0.15191695,0.84346557,0.85209394,0.8199255,0.12635618]
b2d261b4-3b33-4d88-9cd9-7cd634872290	S22	[0.13881178,0.5446118,0.9005388,0.75885814,0.90677625,0.5374381,0.6771336,0.3401837,0.99634,0.060883038,0.0776975,0.43808815,0.3057617,0.9059989,0.7751851,0.14494705,0.620088,0.979791,0.8644771,0.93485886,0.4580388,0.092700444,0.37771878,0.58850294,0.4215482,0.77889514,0.51767236,0.85070294,0.137004,0.1754231,0.3642453,0.1493266,0.352461,0.880851,0.8088855,0.7505584,0.44641486,0.09050554,0.30208796,0.7487654,0.49736336,0.5671273,0.12073264,0.496736,0.3243368,0.31927773,0.08058634,0.97713536,0.9340298,0.9463885,0.7115545,0.1836042,0.15088092,0.5947814,0.78171676,0.96176016,0.83904934,0.71037674,0.20695996,0.6970008,0.45627072,0.64830244,0.11049338,0.09273315,0.8660043,0.03907895,0.40522328,0.57011956,0.5463077,0.44028407,0.80813754,0.65614396,0.29740313,0.6545827,0.8167629,0.26099995,0.8895968,0.73004186,0.7336814,0.66894996,0.53852075,0.6697242,0.23553847,0.7991516,0.95298344,0.069374315,0.27763605,0.42718706,0.35941452,0.011605832,0.81914145,0.5241797,0.18394783,0.89845306,0.1063363,0.9881494,0.96847737,0.0785115,0.79402757,0.99798954,0.6767486,0.8019362,0.7875891,0.25160033,0.7426627,0.27463934,0.09021946,0.97880405,0.19784425,0.7176195,0.39507034,0.06282312,0.9306253,0.85926485,0.35859725,0.028503079,0.6054509,0.6976823,0.7429287,0.8144426,0.29631037,0.054741874,0.8901979,0.67616946,0.7693161,0.97171295,0.44364312,0.06667233,0.031809516,0.62878656,0.57900065,0.120687455,0.36248454,0.6783771,0.026002392,0.8245815,0.6869808,0.44955066,0.07214935,0.5219943,0.28913748,0.26443398,0.6821059,0.66115797,0.946973,0.9203798,0.2434852,0.35822484,0.8538685,0.5716574,0.7961659,0.67458606,0.28633225,0.12382895,0.31482357,0.4195541,0.12205301,0.2946582,0.45532185,0.4836559,0.6953229,0.10676204,0.11474526,0.71073866,0.79860735,0.15619831,0.5188233,0.17979293,0.9590432,0.56101435,0.44727364,0.584735,0.123829216,0.5806128,0.92113686,0.09077067,0.6159213,0.9554034,0.82853335,0.99473643,0.1473155,0.24312505,0.2381954,0.21748541,0.72512144,0.4517537,0.9419911,0.5110148,0.40246716,0.97881955,0.18148316,0.68372655,0.8353446,0.2407955,0.14509495,0.8525547,0.090627514,0.35419574,0.024569396,0.084454894,0.4971539,0.9271579,0.8836024,0.6717098,0.15548843,0.9216079,0.45074114,0.5155946,0.43917903,0.88193274,0.46631277,0.029099206,0.77631676,0.5832111,0.36455348,0.17434353,0.58842945,0.79953176,0.15165167,0.11635347,0.25403678,0.65714353,0.053035833,0.27124304,0.8532927,0.57971025,0.3955383,0.6949078,0.85277677,0.6833074,0.28748778,0.2052665,0.37951672,0.9573663,0.076139234,0.16647752,0.9173155,0.94387925,0.12976418,0.48335457,0.59207106,0.2197223,0.7751946,0.4966893,0.007687247,0.64796084,0.6850379,0.3277814,0.68615645,0.8043388,0.3306233,0.15812767,0.7126498,0.87275535,0.43082923,0.33461133,0.78998125,0.59547746,0.9500865,0.74362195,0.3933257,0.01996316,0.058091026,0.87142396,0.5774719,0.7889588,0.014304144,0.73120457,0.68097943,0.36950874,0.104462855,0.15977271,0.3634892,0.9590248,0.7346741,0.5170667,0.34257635,0.03188074,0.11783994,0.67805076,0.9089187,0.4112126,0.50695795,0.09976118,0.09418238,0.07296319,0.87513256,0.8024079,0.7161288,0.4236075,0.041066896,0.07676949,0.46807256,0.7707967,0.2912211,0.66873413,0.6778906,0.94042057,0.9200404,0.9148682,0.3517264,0.4219253,0.05517007,0.20853159,0.9903079,0.22718155,0.01056919,0.5344558,0.7591299,0.24555655,0.7903992,0.6111131,0.8729328,0.16899927,0.63691527,0.66425234,0.12888053,0.3968246,0.7096352,0.059551887,0.7818251,0.49491698,0.14848271,0.51772237,0.22737141,0.17467944,0.3806409,0.53392273,0.595009,0.36681718,0.09790216,0.20275716,0.9939438,0.2997372,0.594968,0.9970051,0.37570283,0.52977556,0.48042327,0.64111936,0.227191,0.58606863,0.56004846,0.0034042709,0.29758593,0.32964897,0.4387815,0.74895626,0.2529808,0.35100222,0.6856865,0.5515798,0.89876986,0.9790837,0.15352543,0.21733122,0.55369484,0.24904995,0.22921197,0.2333622,0.6768397,0.7517196,0.084842704,0.46109402,0.44085935,0.7219268,0.13108212,0.78902954,0.7444165,0.04702398,0.18198921,0.71228844,0.8552909,0.9375486,0.85075665,0.7411364,0.805241,0.9726745,0.10273356,0.0067646564,0.14670564,0.7710551,0.09860236,0.5706973,0.34638095,0.6308059,0.20959085,0.38380226,0.6237667,0.19421409,0.9095012,0.98070544,0.6894049,0.41045442,0.10845157,0.58254534,0.51928246,0.65864515,0.1388033,0.50584286,0.63862365,0.0116974,0.639489,0.48730627,0.5269967,0.29243734,0.51743084,0.46461385,0.6947578,0.11501881,0.6248612,0.74892867,0.42404947,0.60307986,0.24616133,0.06500641,0.27101952,0.7368764,0.7349516,0.65379345,0.76319647,0.72308993,0.22097132,0.9563546,0.31307486,0.17508422,0.9323583,0.7387155,0.40281957,0.62082815,0.92438334,0.7170294,0.4011374,0.60133564,0.43440253,0.34350127,0.21725641,0.4291239,0.6403602,0.02899837,0.9158899,0.74070454,0.16695675,0.16639842,0.5332329,0.19582774,0.35431486,0.3012782,0.13087636,0.61479527,0.060448587,0.8470124,0.38589144,0.34719887,0.25453848,0.647846,0.5735986,0.51552343,0.37273332,0.5433668,0.37695283,0.29334202,0.9464553,0.037373662,0.61162204,0.4785796,0.89796436,0.4983535,0.22895814,0.88626796,0.4300561,0.32739785,0.21735662,0.80655944,0.6469759,0.82509774,0.03573409,0.9580041,0.86171,0.53464866,0.7596666,0.37100235,0.103775285,0.98856276,0.49981344,0.16698238,0.61790097,0.4018681,0.8902423,0.7937701,0.17303033,0.17674813,0.699143,0.911662,0.1142849,0.4092151,0.12046705,0.9708423,0.0015670487,0.39007932,0.62556696,0.35822317,0.94151545,0.115456626,0.61870056,0.054680433,0.75702935,0.13707101,0.1360289,0.8500958,0.7214523,0.19124913]
ff06e4d2-943b-4807-a3d7-395df26aa888	S23	[0.8584314,0.32356998,0.68016523,0.3169735,0.7927207,0.8356004,0.56050676,0.4401673,0.8244628,0.010659534,0.5679499,0.71125054,0.7774976,0.25957206,0.57070404,0.1725885,0.56334573,0.8065242,0.10472226,0.48208714,0.48621964,0.78421754,0.81387395,0.9318362,0.032229517,0.49928135,0.75267386,0.6749633,0.8947111,0.16916661,0.3295666,0.45465055,0.4579804,0.4904621,0.87437415,0.35811305,0.5600424,0.6581553,0.027037065,0.77614737,0.76164275,0.20337334,0.35359192,0.12326047,0.0698455,0.13828248,0.58467525,0.76685405,0.6719438,0.3079657,0.9363309,0.19173409,0.28864926,0.73467135,0.10903366,0.89801157,0.7139404,0.080591455,0.9930215,0.68293935,0.9535978,0.16339374,0.859095,0.5811084,0.15084806,0.37481448,0.9506413,0.090534166,0.36807692,0.7503501,0.22390512,0.4884328,0.4439517,0.7674308,0.84653765,0.48641172,0.1937956,0.94446546,0.6236186,0.39119458,0.7167355,0.92587626,0.7188124,0.15718074,0.55857766,0.38578323,0.7283719,0.1805223,0.4240031,0.69713056,0.42259622,0.44235313,0.98982644,0.25661427,0.68086565,0.2838075,0.5848385,0.7957931,0.122187555,0.6381392,0.3003164,0.07390589,0.6639114,0.5839644,0.049532346,0.11094264,0.5241727,0.48325413,0.8787012,0.80919266,0.957677,0.72069997,0.39946863,0.8904999,0.012825796,0.035238456,0.48369512,0.8159143,0.988977,0.9901066,0.5744712,0.6000422,0.026412094,0.81223595,0.4960818,0.42672014,0.90897006,0.7564248,0.43990996,0.96932185,0.67621,0.61294234,0.95559436,0.17154817,0.21608767,0.78863776,0.0026286994,0.7611571,0.2695476,0.27993986,0.49826285,0.7206987,0.10640151,0.3990573,0.88766605,0.24120446,0.7605652,0.08528219,0.9560799,0.8742121,0.05029189,0.12084408,0.9928457,0.61840206,0.103406645,0.2232031,0.20183948,0.02917,0.771982,0.78591293,0.95177513,0.9135401,0.44585362,0.43255925,0.2714498,0.3557838,0.94135684,0.6554661,0.8801433,0.69659,0.57778215,0.43240756,0.043480776,0.8923632,0.015486403,0.63471735,0.9097344,0.47215953,0.4226221,0.5123213,0.4223458,0.79332876,0.82423824,0.09730154,0.9531427,0.69736844,0.97704625,0.7536078,0.66890824,0.050033867,0.8335767,0.5584404,0.5650216,0.2731888,0.22132815,0.6658045,0.7782535,0.28330147,0.5509993,0.47899956,0.44603437,0.009573183,0.25742766,0.078126945,0.9093546,0.2823042,0.049477242,0.08925115,0.0580147,0.8615033,0.056223106,0.5014743,0.3776783,0.5966402,0.34431258,0.39811274,0.9572253,0.5193505,0.25087675,0.86509794,0.15746522,0.20848772,0.9870487,0.62908524,0.57040715,0.5489777,0.15284568,0.704473,0.94380635,0.8998579,0.06545013,0.11708164,0.20046073,0.50590956,0.6885522,0.6454868,0.49420798,0.8580547,0.071041055,0.843768,0.48703688,0.8287578,0.25348777,0.9967393,0.39427477,0.53164566,0.27002692,0.893942,0.43312985,0.5474635,0.9078047,0.29623026,0.75631094,0.99045205,0.59389246,0.68462515,0.3849221,0.24801093,0.15800248,0.9576318,0.9058362,0.015024568,0.21069753,0.47298193,0.5032381,0.8115455,0.98128426,0.11942284,0.58248216,0.5995309,0.71541995,0.96050525,0.8469002,0.64835393,0.4192338,0.09482266,0.652201,0.6367329,0.17731258,0.1553936,0.061566148,0.35453996,0.34042656,0.021094313,0.5807078,0.8647541,0.64826447,0.572913,0.5232441,0.5856064,0.2491016,0.96771526,0.22065628,0.8761029,0.07705142,0.6927481,0.59952295,0.09467276,0.9979211,0.114078686,0.1864776,0.9791604,0.7992438,0.18455912,0.19034685,0.46628729,0.011250687,0.3260945,0.5027377,0.16859184,0.27529603,0.10251221,0.55140686,0.2088314,0.13445573,0.36279836,0.19544193,0.836205,0.3972611,0.278822,0.022160452,0.4418079,0.1772338,0.5300696,0.98431855,0.6197583,0.62641245,0.9235883,0.15956745,0.16741924,0.33000726,0.031957764,0.39783654,0.3057037,0.47151816,0.19727537,0.030943662,0.65350896,0.49635997,0.8479917,0.6809785,0.30805433,0.27882206,0.3183315,0.057883803,0.4760439,0.027698614,0.4860949,0.8210629,0.4598237,0.17455788,0.96648294,0.12772341,0.56311715,0.6519008,0.5848614,0.9977402,0.780859,0.99050695,0.85914564,0.43533814,0.5313219,0.21372655,0.3337544,0.51471394,0.3505971,0.9875596,0.40736955,0.020092802,0.43473104,0.60189456,0.31475356,0.5880209,0.030161297,0.9328075,0.9202538,0.38111815,0.5965001,0.30594644,0.4344091,0.55102533,0.7910904,0.8183863,0.63523924,0.6911622,0.25093675,0.81072354,0.4011766,0.26388332,0.65073556,0.32904845,0.38891056,0.18849328,0.030470207,0.2274791,0.87144905,0.53786135,0.44293347,0.40921807,0.7388046,0.09783452,0.74779177,0.69261265,0.5536544,0.8661087,0.88085085,0.37094748,0.7919734,0.19375305,0.71529984,0.49685842,0.918503,0.5170682,0.35803267,0.43571794,0.5021596,0.972721,0.07384836,0.43798262,0.10265242,0.7830508,0.86638796,0.15502737,0.38678005,0.054723017,0.2066353,0.81169826,0.6901525,0.026140042,0.9957288,0.6515971,0.46060348,0.9013058,0.93528205,0.40588132,0.44884884,0.27055284,0.66971743,0.9966368,0.9903694,0.8917767,0.20524827,0.32579726,0.66146976,0.78661054,0.22449993,0.6957185,0.76867026,0.6641914,0.1619978,0.48256457,0.07652739,0.28996205,0.71398497,0.3253161,0.3317782,0.5383313,0.8304613,0.7388274,0.3536382,0.22067493,0.70003563,0.6812773,0.44421923,0.97328204,0.4725742,0.49019098,0.848009,0.9386857,0.05359526,0.1453841,0.97777045,0.82630056,0.94076854,0.62578297,0.3248759,0.7693073,0.66449016,0.9672339,0.85665244,0.08667142,0.31614015,0.104632825,0.49762568,0.7108472,0.40758067,0.4902133,0.43834144,0.35556352,0.29479265,0.8318298,0.5893074,0.47029182,0.4222851,0.8178352,0.6187548,0.012386704,0.2533327,0.2363039,0.98788315,0.067562416,0.73968166,0.5238221,0.7617454,0.4108434,0.29016408,0.52214557,0.6529271,0.25471035,0.6457031,0.44418192,0.9710753]
22309dfb-85ce-4221-beda-c12796788357	S24	[0.7445889,0.5268192,0.27645463,0.6029744,0.5072593,0.51586163,0.65505105,0.84913844,0.94633865,0.693298,0.34579894,0.8967399,0.5424509,0.1749243,0.2881607,0.3165347,0.53179145,0.6610448,0.58182764,0.005078563,0.007647784,0.3790703,0.15090893,0.26921037,0.03345583,0.8463589,0.31840876,0.6985938,0.8866667,0.25859445,0.7942543,0.93831855,0.78347415,0.5360584,0.20613435,0.17538385,0.48650235,0.36031365,0.4660881,0.08606099,0.13062374,0.5327696,0.15127179,0.45838794,0.5892241,0.682055,0.9710453,0.0065180394,0.44430512,0.32895333,0.20259671,0.8383896,0.078675635,0.43423432,0.76745325,0.976179,0.6566137,0.48200113,0.27635902,0.3981192,0.7128678,0.4648389,0.8368706,0.054190233,0.11093587,0.3088926,0.6349497,0.12679899,0.2436043,0.23454249,0.52085334,0.650661,0.060214575,0.27858096,0.65906787,0.15029433,0.6661398,0.23284954,0.985921,0.03763376,0.12414354,0.6501407,0.33697242,0.07566871,0.43323502,0.34457248,0.54115725,0.1648085,0.13273846,0.97105473,0.7972966,0.15989193,0.93742186,0.8306537,0.3249291,0.54827553,0.010776293,0.637628,0.06604446,0.7373616,0.21454759,0.88281184,0.039571937,0.19595158,0.39370126,0.71097714,0.027618257,0.20909144,0.37793705,0.272619,0.23321721,0.6534489,0.9808805,0.92473936,0.3838305,0.7911805,0.10794843,0.25911015,0.16531274,0.35844424,0.17337465,0.62974775,0.44636247,0.34511617,0.04996962,0.66923696,0.6293479,0.15054771,0.4236546,0.7992485,0.39974356,0.5987973,0.40844393,0.23388346,0.5670263,0.50594866,0.72165036,0.48208505,0.09449252,0.7816334,0.98570323,0.7126253,0.9664782,0.5606946,0.012044337,0.79375166,0.8165974,0.79218173,0.88009167,0.08385641,0.90683484,0.3869772,0.6320226,0.12584044,0.43776727,0.66493607,0.030002195,0.10984572,0.91914034,0.6154687,0.08647052,0.5123564,0.542976,0.5493299,0.8203254,0.9857415,0.008867111,0.1328862,0.58432364,0.48353177,0.9366451,0.7604598,0.07301905,0.6882712,0.86422217,0.7206495,0.446419,0.5922773,0.40968397,0.39064565,0.27938038,0.80284315,0.5282707,0.6110001,0.6406277,0.589134,0.51525223,0.74821323,0.36387196,0.6544457,0.19804212,0.58539605,0.8576193,0.51785064,0.8360452,0.8336725,0.6649634,0.86433834,0.3309871,0.12987909,0.8463823,0.090750806,0.3432711,0.39630437,0.9607599,0.7480563,0.92272264,0.6242895,0.8167372,0.6009509,0.43204638,0.18501401,0.5597024,0.90241575,0.65095276,0.8599311,0.45892793,0.041891415,0.054104984,0.83691835,0.7233301,0.6677688,0.8160836,0.18258621,0.6577949,0.5511015,0.17579688,0.5115405,0.92785335,0.5399058,0.80111,0.07170396,0.34615907,0.32724413,0.37859148,0.7113868,0.06515372,0.9652663,0.2501667,0.658212,0.25535274,0.5285532,0.2754688,0.45425966,0.6563929,0.23226205,0.78578806,0.74146575,0.28259966,0.64598703,0.110639974,0.9138837,0.3768615,0.89120024,0.11323922,0.8080102,0.45802122,0.045241326,0.55050063,0.14031409,0.26211876,0.8999298,0.49461362,0.79354906,0.54648477,0.35761428,0.42594838,0.16504496,0.9459003,0.7142212,0.9418215,0.41803044,0.81125575,0.7316426,0.27237874,0.9424249,0.47558442,0.7628904,0.79177016,0.9200307,0.59495234,0.5739007,0.18971536,0.09641484,0.2552904,0.64264345,0.028099863,0.58231777,0.8767739,0.41447878,0.8917258,0.09888693,0.9461893,0.29323733,0.24419846,0.81358755,0.81967396,0.2922396,0.92468935,0.6730621,0.6001929,0.73372483,0.3055761,0.19745813,0.66488606,0.36997753,0.45333,0.9560478,0.53065735,0.49987474,0.048780434,0.702198,0.7904363,0.9075723,0.22826692,0.41294435,0.5925217,0.7216408,0.44801852,0.16490744,0.38688263,0.70208716,0.23935157,0.7139009,0.7034066,0.62516516,0.9697276,0.2678697,0.7411517,0.7474814,0.35754785,0.3236941,0.14970717,0.3959838,0.057390507,0.68583304,0.8710508,0.91400146,0.8254592,0.67788756,0.79641813,0.620687,0.2536194,0.94256634,0.29006177,0.10555999,0.86097026,0.21361828,0.6331418,0.44455475,0.09263721,0.8400789,0.87551683,0.4601088,0.8900268,0.6119005,0.7625095,0.02517414,0.48726526,0.5405592,0.17723827,0.27513626,0.25344723,0.3609962,0.326911,0.95108724,0.27017426,0.8481454,0.4019721,0.26251674,0.5238165,0.8389591,0.43613306,0.95985985,0.23978393,0.9398225,0.64864355,0.58322996,0.48293352,0.17229062,0.46613073,0.52587426,0.35496244,0.91136396,0.8027973,0.9291609,0.12848134,0.99367887,0.6095355,0.014039751,0.49848574,0.07914695,0.80007833,0.18986225,0.943924,0.91099006,0.91675764,0.88146377,0.25270826,0.72394025,0.15669766,0.28977716,0.25966406,0.66432655,0.30890316,0.34337476,0.7620081,0.31453294,0.4554048,0.6164811,0.694703,0.21526623,0.931843,0.6573072,0.036159016,0.7366109,0.64867055,0.9868916,0.8178167,0.22650044,0.66856116,0.09186103,0.09616167,0.3808401,0.6325045,0.6018593,0.46793753,0.75048393,0.99786437,0.7075975,0.9261085,0.4003527,0.516369,0.72162014,0.40485203,0.7464566,0.5336355,0.9569047,0.77916485,0.5326276,0.73516333,0.018501434,0.55877316,0.48730472,0.8107383,0.36604998,0.2765173,0.7867472,0.7311306,0.7340363,0.73597485,0.45110184,0.41498178,0.6819924,0.20892681,0.6598416,0.99175537,0.2394012,0.73396915,0.5718333,0.17237686,0.4694863,0.2552467,0.053514637,0.2890574,0.098332524,0.16259041,0.7478626,0.22384953,0.2672553,0.72333056,0.9445225,0.28058025,0.5152043,0.3397182,0.019190604,0.061509043,0.32392862,0.82936347,0.2170532,0.10610911,0.60091704,0.8545429,0.48198828,0.70739704,0.6610749,0.46222273,0.6638635,0.15626322,0.42074835,0.48046863,0.98350227,0.34052628,0.41740584,0.81834036,0.8249695,0.8522003,0.4462986,0.8358138,0.8406098,0.022742968,0.3381397,0.9526199,0.21427527,0.77193683,0.5311442,0.7936851,0.507966,0.15239793,0.70802265,0.2495969,0.78732014]
1eac1149-9c04-4934-93f9-74f2e7fc7fd3	S25	[0.88502514,0.9766892,0.82400656,0.37586662,0.38734365,0.2564453,0.63198763,0.99159384,0.60034305,0.8240486,0.7151737,0.36368,0.33738923,0.5401934,0.9473431,0.71939164,0.7588701,0.55591625,0.72263324,0.48744765,0.39729246,0.21641454,0.12774524,0.020584483,0.17550215,0.19505888,0.8736169,0.028323708,0.9514847,0.98401177,0.4111382,0.8201145,0.56515896,0.14925383,0.7757765,0.6158804,0.2840404,0.9852739,0.81914043,0.16408628,0.6642927,0.73147184,0.92967546,0.58804554,0.6866828,0.14207071,0.29240564,0.70326144,0.8615039,0.4628553,0.556084,0.2955404,0.49250734,0.958584,0.24849553,0.7892391,0.87398994,0.7272619,0.04036842,0.13281228,0.4071661,0.012124624,0.91941774,0.68760574,0.8602122,0.94028103,0.87258625,0.07668112,0.9457222,0.8236694,0.4289728,0.17531833,0.49720842,0.967177,0.49804688,0.6592156,0.9291122,0.7532291,0.12508497,0.03888247,0.64462715,0.11759015,0.1608781,0.641445,0.9640513,0.4317525,0.8737865,0.20435928,0.8631613,0.3694365,0.10315327,0.04788596,0.8479581,0.5921909,0.89883167,0.46404153,0.12893161,0.7889971,0.31265634,0.112781525,0.028777108,0.6328482,0.16840798,0.58249974,0.4199163,0.6379444,0.6939464,0.712736,0.94200057,0.6793372,0.85928035,0.77593553,0.09630328,0.15652254,0.37459126,0.665068,0.9420383,0.9293605,0.22056417,0.5662106,0.3370864,0.5735083,0.95091516,0.09375078,0.9641258,0.5710045,0.5545406,0.5887561,0.30949497,0.59333277,0.61919385,0.8170384,0.35023698,0.27560064,0.90818655,0.36104265,0.102133155,0.6041961,0.32303903,0.93996,0.31998476,0.91486514,0.7442624,0.18979885,0.81326914,0.36355886,0.9028857,0.6198427,0.8224186,0.78661823,0.21832699,0.4397654,0.2860736,0.97985446,0.19796549,0.23640801,0.09535535,0.8544454,0.5567625,0.32877323,0.2517661,0.44324952,0.37842873,0.4819194,0.0045124753,0.1841891,0.5237303,0.38785276,0.9779477,0.2710445,0.25872496,0.82447666,0.90797865,0.25850195,0.9548137,0.355424,0.6706155,0.056293588,0.04265655,0.24068321,0.6631316,0.5848294,0.5773419,0.014840245,0.61249655,0.5327414,0.82977045,0.07904874,0.91569346,0.5518712,0.84136975,0.6984296,0.24011938,0.47694838,0.21065055,0.65739954,0.8460824,0.79409057,0.9877327,0.9310164,0.60793585,0.61281186,0.051459182,0.8474326,0.44545555,0.6695469,0.8203371,0.11973571,0.0011225998,0.068769425,0.113747254,0.029307446,0.8079595,0.18566485,0.5092503,0.21080996,0.18337883,0.33674,0.85310334,0.08577582,0.55050415,0.41603494,0.40458506,0.8612595,0.06709053,0.9629915,0.63322806,0.8004735,0.26759207,0.02059368,0.5614545,0.49738583,0.9491196,0.34064674,0.8879485,0.37831494,0.40691882,0.18614836,0.3894513,0.5359849,0.3528536,0.18383905,0.5326986,0.7532804,0.90384865,0.48437217,0.7238245,0.90032727,0.1348095,0.042242464,0.5151386,0.5046838,0.7877536,0.116762,0.7145614,0.12479505,0.5556635,0.48753226,0.07565995,0.20245077,0.90873665,0.87106156,0.02534243,0.7855398,0.054084823,0.89093137,0.45051634,0.663469,0.48228362,0.4010062,0.39222485,0.30927783,0.43581462,0.77210957,0.31438234,0.7052231,0.34119606,0.4619467,0.5575717,0.9686365,0.73483646,0.30444118,0.22526988,0.14578675,0.040593803,0.6403609,0.115676396,0.23959485,0.8959757,0.38216496,0.9156874,0.34277204,0.17972322,0.949267,0.1694709,0.95733505,0.9857623,0.87232906,0.108620286,0.48580194,0.5215794,0.27542835,0.013994348,0.8986843,0.41445076,0.57886934,0.24752125,0.018654259,0.2330654,0.39719942,0.671036,0.05217398,0.24314578,0.17604801,0.7174923,0.9684655,0.4354069,0.7682181,0.9772527,0.37439877,0.08091796,0.2908292,0.74553365,0.14354493,0.034642614,0.06271619,0.2975997,0.052994367,0.69222325,0.27911872,0.932993,0.6093343,0.23561779,0.09878005,0.58498925,0.75979805,0.41268823,0.06539486,0.04841272,0.7763463,0.53388166,0.3750175,0.48195696,0.7509307,0.71538854,0.101499155,0.88557386,0.33907872,0.83097756,0.5917457,0.40020064,0.9615682,0.5784695,0.0032950803,0.9086872,0.5639808,0.39035544,0.23183097,0.07243871,0.15898393,0.836345,0.27167523,0.98443705,0.17963658,0.61875355,0.7394153,0.02391096,0.113030806,0.7619833,0.30108047,0.54687244,0.34814888,0.5505854,0.040415235,0.7501145,0.26696327,0.7824711,0.7912537,0.29408753,0.91120774,0.1966444,0.43822038,0.98295027,0.5318775,0.791125,0.05368104,0.73741186,0.88819563,0.42958272,0.54181564,0.71542543,0.2783611,0.550789,0.46845174,0.41894093,0.9991548,0.29824537,0.7355254,0.5682304,0.12431843,0.006374201,0.69998336,0.06896533,0.8377354,0.6930565,0.461845,0.8713765,0.5743477,0.010814565,0.1082023,0.42088768,0.9222759,0.054633096,0.7723519,0.0010110614,0.71052617,0.9026865,0.23313075,0.31802583,0.88592607,0.9573848,0.2683172,0.46891662,0.6869456,0.9597406,0.40535614,0.36266193,0.5998476,0.9120436,0.39302102,0.031747706,0.6514025,0.028227005,0.31220096,0.31642085,0.39244533,0.53253776,0.3264613,0.74495965,0.28896165,0.009911038,0.15776803,0.40894836,0.8558724,0.7694881,0.9920595,0.1684647,0.6729493,0.84313023,0.18965292,0.15100172,0.752657,0.55767196,0.6550005,0.110957146,0.5596226,0.0490892,0.6841898,0.0988143,0.0064674644,0.5461232,0.014732869,0.9589684,0.34311688,0.9736541,0.77060366,0.77178806,0.8989152,0.6060004,0.429059,0.19273806,0.4319808,0.8496233,0.50704324,0.9236661,0.9055475,0.1697552,0.31062165,0.987717,0.70187736,0.27765006,0.262506,0.33899763,0.00737217,0.38251576,0.9731599,0.92502946,0.7319217,0.9794909,0.409138,0.19751823,0.8460633,0.3327282,0.57495695,0.90826774,0.411665,0.63959354,0.37799847,0.16513363,0.938379,0.9964548,0.68583554,0.66701204,0.79858977,0.43224677,0.5589868,0.20398116,0.18753314,0.30299374,0.63239324,0.19449313,0.2612451]
50acc0da-9191-4e0d-b2f1-f95f49e85c8a	S26	[0.60272884,0.21309881,0.5668332,0.41462356,0.25913787,0.4902971,0.45087916,0.63311136,0.99290127,0.9011824,0.8683515,0.95560914,0.8587577,0.1710287,0.33226594,0.87720656,0.17825823,0.91936326,0.15426798,0.22038816,0.3199843,0.066623054,0.3704272,0.78580874,0.013780225,0.33685222,0.26071322,0.5963301,0.53504986,0.27756387,0.8315115,0.63983834,0.91325665,0.63752145,0.650537,0.08456533,0.6857751,0.24542822,0.96801114,0.63902247,0.90644765,0.574173,0.92533267,0.6300307,0.15348421,0.5795933,0.9054857,0.48026186,0.8935994,0.454763,0.98428833,0.8886807,0.8731566,0.54154384,0.12463009,0.08537609,0.15060894,0.95568377,0.31568897,0.38525057,0.34051672,0.1338828,0.051921126,0.22922221,0.665618,0.648592,0.3777817,0.48777628,0.6580474,0.2650716,0.0998537,0.13248473,0.21965744,0.8987442,0.051618304,0.48036566,0.17028257,0.10465503,0.972171,0.85013604,0.69796515,0.31265023,0.7930581,0.8221736,0.45998722,0.7615698,0.008636404,0.28241655,0.99263734,0.44158632,0.15169473,0.20719549,0.648938,0.34854382,0.991357,0.32059148,0.17645004,0.4009492,0.04604703,0.29966563,0.24786285,0.22957398,0.7644096,0.0807305,0.6732443,0.034673713,0.010437365,0.5639488,0.027469212,0.36062282,0.8983207,0.31477958,0.42990574,0.5293532,0.32549837,0.8795769,0.64434266,0.105730444,0.76710904,0.48209518,0.1115403,0.23839912,0.1452886,0.31460008,0.10359023,0.7386148,0.8869627,0.004125861,0.10019305,0.9029642,0.62923914,0.85422486,0.5372809,0.7106264,0.14598994,0.4374314,0.78120553,0.57452905,0.74400723,0.2754512,0.8954965,0.35388905,0.36559066,0.23975162,0.2947127,0.12153846,0.3539108,0.535384,0.1217629,0.1570185,0.07601227,0.9236191,0.22954051,0.48099923,0.14231698,0.708831,0.08954315,0.8139613,0.87790906,0.07727644,0.13923237,0.9370358,0.81145513,0.7084638,0.7212394,0.5912891,0.8384418,0.0275378,0.319898,0.6112832,0.5914446,0.5182609,0.27106628,0.29120508,0.49888653,0.75436074,0.9418649,0.350844,0.47259888,0.5610298,0.46380866,0.19177449,0.271186,0.1914588,0.59459126,0.81360495,0.61970717,0.7876408,0.19943069,0.19534735,0.2036145,0.29974037,0.7360742,0.7804994,0.7472405,0.09720543,0.83408856,0.59782594,0.8322058,0.54293436,0.55700094,0.8185266,0.81181663,0.88311285,0.6407808,0.08356631,0.37120947,0.37405798,0.19214706,0.5698919,0.87318426,0.5616981,0.3569802,0.09039657,0.8516473,0.9017105,0.96436054,0.6074821,0.43625888,0.736494,0.7710724,0.18510994,0.4515792,0.9948433,0.92431664,0.74823076,0.4178662,0.16828291,0.81416804,0.763347,0.81206405,0.13386223,0.32456836,0.829421,0.8054146,0.11412178,0.12651533,0.6492997,0.8604507,0.66335547,0.12402147,0.028740289,0.14448377,0.057893172,0.28205162,0.12425828,0.11288432,0.15991896,0.88022876,0.86872584,0.6474273,0.3588336,0.51542246,0.36966825,0.72562116,0.020148516,0.24124245,0.29854834,0.23036295,0.5814733,0.65700126,0.3056053,0.72374165,0.6755864,0.60218877,0.29186788,0.5001716,0.35691702,0.49281585,0.06096648,0.8609043,0.32203814,0.88307154,0.4412841,0.45077857,0.1087931,0.31175205,0.88398373,0.82363164,0.033516247,0.12041657,0.54557234,0.11251115,0.47902298,0.30034834,0.8206467,0.8952468,0.5348382,0.29129055,0.6380669,0.40282777,0.28684834,0.62475306,0.07999825,0.6622843,0.012859542,0.11220547,0.17598261,0.9374049,0.373928,0.44123927,0.048757467,0.42770728,0.11969282,0.7058872,0.62552685,0.6688427,0.4520804,0.133202,0.76325244,0.37046036,0.25343293,0.9273255,0.19786519,0.18761803,0.16295458,0.11749026,0.32424882,0.9075766,0.7815296,0.30917335,0.03319393,0.083270475,0.54817915,0.035680305,0.78340113,0.2001442,0.074456625,0.6312274,0.23261592,0.67490435,0.8547536,0.24973755,0.022651475,0.96273005,0.6223736,0.8413766,0.7501741,0.84735405,0.78275144,0.5551702,0.7728556,0.5224599,0.078896955,0.34305814,0.77430207,0.61323667,0.42343682,0.03179597,0.75550985,0.062036354,0.475989,0.93793523,0.93776643,0.27752513,0.8046553,0.18083937,0.5755852,0.84364706,0.4893396,0.382943,0.5294633,0.8310815,0.31741738,0.9169801,0.1832835,0.74008113,0.64161503,0.3588665,0.422315,0.13268328,0.68403745,0.43960872,0.79259175,0.50345355,0.8838474,0.6296317,0.17342615,0.48311695,0.10015573,0.913455,0.08355017,0.74414104,0.7284866,0.8979668,0.74114436,0.021003693,0.1743269,0.93457836,0.3744713,0.6278095,0.5173481,0.8441012,0.18927298,0.47589937,0.6842461,0.59455514,0.8532349,0.55554223,0.34478557,0.7356376,0.70470095,0.26927963,0.7810642,0.10443545,0.26799634,0.2711781,0.42389792,0.4332252,0.09678755,0.5151221,0.14729643,0.7699084,0.27451646,0.037535757,0.36721098,0.7602175,0.22433537,0.41469026,0.13027397,0.5756878,0.04336216,0.22161679,0.15650204,0.5587764,0.26026127,0.13148662,0.6746984,0.62084645,0.20996228,0.24552444,0.8204414,0.22177045,0.8699131,0.46847695,0.19670884,0.4380252,0.034207284,0.7108316,0.7041002,0.57101625,0.4901339,0.36843663,0.37629378,0.45240748,0.66717976,0.84934837,0.059187472,0.0045223306,0.8567046,0.9039386,0.29849526,0.8026092,0.87691087,0.20212017,0.74721676,0.2947155,0.021689322,0.6596786,0.4347824,0.87481934,0.36734858,0.44082522,0.777115,0.49936315,0.84248453,0.4988609,0.23230277,0.63288784,0.8085781,0.10968963,0.30750087,0.61895674,0.07290754,0.02664489,0.64844173,0.72170603,0.5596218,0.040907096,0.25822115,0.5254778,0.95872784,0.022245243,0.8947155,0.24564984,0.3531334,0.2971504,0.87460804,0.6628318,0.925707,0.8293117,0.32176962,0.11530357,0.5567301,0.716789,0.12169464,0.18324552,0.0470442,0.14998521,0.8234924,0.98672503,0.2797892,0.59387106,0.91597736,0.16680317,0.24789952,0.16977131,0.5732995,0.2581564,0.35621348,0.7562197,0.09597892]
d9445cb8-2348-433e-ac8b-3af85feb6469	S27	[0.15726633,0.39009166,0.22234395,0.5718196,0.22928727,0.09500777,0.68080693,0.9991355,0.23802412,0.8239797,0.12693968,0.960085,0.98332924,0.5168998,0.109173924,0.57596356,0.54945683,0.69498426,0.11477966,0.25521916,0.096053936,0.34885117,0.45851865,0.009383459,0.4362637,0.73123723,0.79410726,0.46697047,0.9267024,0.119869284,0.65424603,0.97550267,0.9353344,0.46273234,0.9130939,0.41701326,0.84374744,0.5586107,0.30767968,0.91531384,0.97433233,0.49944088,0.7527402,0.88496816,0.599217,0.9707429,0.7076177,0.785642,0.5118389,0.66702175,0.6093322,0.33293313,0.5279798,0.06628138,0.5095252,0.61622965,0.8379446,0.8385154,0.8208569,0.8537009,0.03548604,0.31581894,0.43123397,0.57256794,0.91429657,0.86069286,0.21709418,0.12242872,0.5480391,0.83265316,0.83868355,0.98566216,0.7362783,0.5906923,0.15838963,0.61800617,0.24082056,0.7236489,0.52330613,0.042184856,0.59794956,0.12295442,0.1898308,0.26130804,0.33732307,0.3375,0.7322052,0.84780973,0.44753778,0.42663187,0.77847725,0.55524427,0.50333476,0.6436924,0.6382759,0.4756187,0.32969597,0.2669978,0.5267118,0.5156066,0.44616163,0.5164272,0.6777605,0.3326978,0.27770442,0.18791047,0.33716178,0.918368,0.5057438,0.6592706,0.6783724,0.9490324,0.37239748,0.58023393,0.014875549,0.931062,0.87935615,0.47204828,0.9109318,0.5558187,0.4112666,0.46589318,0.71892035,0.5627002,0.25350899,0.120096065,0.6298083,0.5221007,0.3012458,0.6204842,0.027417943,0.1821141,0.6479425,0.9915819,0.52459157,0.34743464,0.6570352,0.8959729,0.7032121,0.6747184,0.3656071,0.62248033,0.87233555,0.33653638,0.8681168,0.6988394,0.2629269,0.43428308,0.22680844,0.7129032,0.73172575,0.104422584,0.7606604,0.2819525,0.29739073,0.56537956,0.4689295,0.7893824,0.42533204,0.5101142,0.18428941,0.57300264,0.91268474,0.01929256,0.4042579,0.49017558,0.83078474,0.8838135,0.17105153,0.058597684,0.7863899,0.51177627,0.18268205,0.39722884,0.5754109,0.7349104,0.9518901,0.2555222,0.8703178,0.8556477,0.7129776,0.61682916,0.43407017,0.9926171,0.38873968,0.94375944,0.11975873,0.1907273,0.7400071,0.6102401,0.94856334,0.24110347,0.5234737,0.9813676,0.66440296,0.5437715,0.46268538,0.800627,0.7799856,0.37069663,0.36779377,0.24992375,0.18339965,0.27720878,0.277056,0.7199454,0.1950931,0.37857187,0.31809583,0.5511199,0.57683086,0.57927835,0.0936586,0.2876039,0.053980224,0.73021513,0.1561648,0.67640036,0.33431846,0.9926795,0.2900885,0.69365853,0.46345899,0.6005163,0.16348173,0.92926306,0.7777101,0.19768275,0.51242304,0.23567463,0.18935327,0.9026962,0.20533748,0.46499062,0.28414682,0.2975396,0.7330757,0.037570488,0.25987533,0.30529007,0.5526281,0.19343512,0.9583268,0.17561044,0.6997712,0.1881199,0.66420084,0.49552497,0.98848516,0.3748955,0.7825955,0.6525606,0.25663775,0.44173846,0.6476716,0.98635626,0.12484555,0.40510225,0.8215979,0.1934362,0.6063346,0.50924724,0.47512206,0.54044974,0.69883436,0.16356543,0.955708,0.17822498,0.10482378,0.4474761,0.7137398,0.18541986,0.13073786,0.073073804,0.008471084,0.26877972,0.22250001,0.27680358,0.96561193,0.18264952,0.5476632,0.54116905,0.2560475,0.06148276,0.8574774,0.5313415,0.60975415,0.243686,0.7770657,0.3328998,0.66737,0.52075726,0.9355318,0.482559,0.85070527,0.5601832,0.52242064,0.2294677,0.19588104,0.61573505,0.8002178,0.6770107,0.08265426,0.4192367,0.23200126,0.98594064,0.83486414,0.7887945,0.902685,0.08782432,0.14455988,0.2871676,0.8693481,0.052281626,0.46558687,0.2671178,0.18997565,0.9990121,0.40156904,0.48650488,0.18190593,0.90359944,0.7323018,0.33924338,0.10175604,0.16177706,0.6178079,0.6367986,0.3094595,0.74784833,0.1017846,0.14882532,0.052604817,0.7148635,0.32552153,0.21331915,0.1627706,0.2454241,0.9793846,0.31352922,0.80289143,0.84872013,0.1163952,0.13488145,0.8557311,0.80048543,0.47964263,0.09809026,0.44809064,0.8877144,0.0734177,0.4431782,0.22153907,0.4287365,0.66558665,0.34101197,0.039540388,0.15680349,0.2812103,0.46178532,0.40363002,0.43327194,0.038841326,0.82860047,0.8345902,0.024700414,0.445127,0.8331754,0.80483115,0.9698406,0.28840417,0.22091405,0.8003332,0.6234923,0.16533151,0.57298756,0.73249304,0.21239442,0.017594874,0.16083166,0.8942193,0.70699346,0.89341414,0.9732884,0.9018862,0.119370855,0.8081984,0.45107093,0.80110765,0.57303154,0.76161736,0.8198296,0.1909912,0.5618712,0.73125637,0.30671287,0.509361,0.53679085,0.2995594,0.053946692,0.061472207,0.004830696,0.050585456,0.14947803,0.5128208,0.021511909,0.22318812,0.9270465,0.29988396,0.96937084,0.47797644,0.07448018,0.7446704,0.56003606,0.27246615,0.60854733,0.23896752,0.14186443,0.25426275,0.7995335,0.87441045,0.22279827,0.85413814,0.21131833,0.8304943,0.21021922,0.10167978,0.97770417,0.82014585,0.2849891,0.50879824,0.42816982,0.6606618,0.8973066,0.15019259,0.101155646,0.9115285,0.2619692,0.83138025,0.7998159,0.48955867,0.17720386,0.42748767,0.4054741,0.8975909,0.30823705,0.06013811,0.66779035,0.17571147,0.61074644,0.47435266,0.267873,0.46930516,0.6659674,0.62217677,0.03061759,0.25020936,0.8142063,0.62938976,0.112352744,0.13327542,0.35366172,0.57295585,0.106253125,0.67926097,0.3168116,0.4772873,0.9497,0.36320648,0.3077822,0.95390755,0.21957485,0.87796324,0.5935989,0.49759015,0.308707,0.04404785,0.35302043,0.73069274,0.24546093,0.0858491,0.7720626,0.2787306,0.5389032,0.81883985,0.6521185,0.41355866,0.37934825,0.77731234,0.5195748,0.3751209,0.7720209,0.3148215,0.92878485,0.44451645,0.53897625,0.8165822,0.23203494,0.74588424,0.29163155,0.5366241,0.31048143,0.5307918,0.09091407,0.6104857,0.33670595,0.91025096,0.94041777,0.1358728,0.7511016,0.7684656,0.77834594]
a416164c-78e4-46dd-8038-3c81b336190f	S28	[0.57640666,0.15079659,0.25546765,0.8453047,0.37569204,0.6911936,0.8862907,0.80050796,0.60185593,0.6371769,0.7845001,0.104714654,0.93059415,0.59407955,0.94506896,0.7628553,0.26602384,0.8533361,0.8706041,0.7273418,0.952727,0.30426252,0.15015176,0.27085412,0.99539405,0.24383569,0.5545209,0.30947602,0.6319415,0.94159234,0.44747654,0.24482219,0.8033379,0.040730063,0.2682033,0.97553396,0.85554075,0.90951777,0.4781677,0.060350407,0.07563205,0.47744086,0.2616379,0.7679541,0.40256953,0.3727759,0.27463233,0.12301388,0.9904591,0.9706263,0.8434229,0.3233047,0.30677003,0.71316296,0.9634888,0.07739151,0.9889177,0.40158716,0.18732369,0.12689418,0.34210065,0.308604,0.035543483,0.17726463,0.5413517,0.36900604,0.4216974,0.13027559,0.12149889,0.46669072,0.112712435,0.28032738,0.22795798,0.06860091,0.60089344,0.9113091,0.65604824,0.9946906,0.46268827,0.09805644,0.20593049,0.43265167,0.37525034,0.1400908,0.89437443,0.1720577,0.19043113,0.6853864,0.36399683,0.80216044,0.0800785,0.6586098,0.5455144,0.60124826,0.94075036,0.7094492,0.9637865,0.63368106,0.21704781,0.2150729,0.69022834,0.17968279,0.6630536,0.6921219,0.5161199,0.04356216,0.9878747,0.39694646,0.29247636,0.40683132,0.29428777,0.028672012,0.0138858985,0.8293448,0.17961746,0.282647,0.18006746,0.40446126,0.24450924,0.96886736,0.5012903,0.74502957,0.080929965,0.30330175,0.89575285,0.4783422,0.4907674,0.8399843,0.71589744,0.8760017,0.53189635,0.7480926,0.87758803,0.570389,0.5652711,0.4692093,0.032895252,0.24403422,0.88373303,0.86688876,0.99752355,0.98874986,0.16871881,0.09599399,0.4868566,0.687088,0.15861554,0.75550854,0.84739155,0.5831928,0.31565157,0.40995103,0.29403663,0.24186322,0.38688362,0.28693128,0.23456493,0.4648073,0.92458606,0.5838741,0.1290373,0.8442168,0.4812362,0.39696202,0.56294906,0.5901072,0.44966102,0.4932479,0.2786811,0.3281604,0.26834765,0.6627175,0.45263,0.4338048,0.2463252,0.08640901,0.5326541,0.668773,0.192658,0.92637515,0.67706335,0.70326954,0.3548275,0.113954,0.27624047,0.34196106,0.25807467,0.20532067,0.9340877,0.7733218,0.62926835,0.2520215,0.71269095,0.21153486,0.6594631,0.90153664,0.78443456,0.76509726,0.48910582,0.99998266,0.86488885,0.66501665,0.3120463,0.38647252,0.6708661,0.70728296,0.8937353,0.8042701,0.693276,0.9217302,0.13981661,0.3994369,0.3763144,0.5334765,0.80309916,0.34780082,0.20838377,0.9988102,0.32977253,0.7842461,0.03224099,0.39037082,0.806466,0.15495424,0.8049048,0.4660526,0.44651166,0.5603133,0.085237496,0.5019595,0.82577306,0.6153285,0.8180493,0.8846665,0.18391395,0.28732258,0.80509084,0.22257882,0.77163,0.52425075,0.017251465,0.48746392,0.6411887,0.8789912,0.60505027,0.24398492,0.25856683,0.59616095,0.09866477,0.9639968,0.6420364,0.31552124,0.12176576,0.8144243,0.11805861,0.45707685,0.470321,0.21990067,0.118783,0.18361692,0.9175267,0.91839534,0.55671364,0.3833502,0.19090909,0.087549634,0.35608485,0.8845763,0.17029518,0.6449843,0.31763875,0.7060746,0.072886705,0.7555944,0.5660112,0.9713224,0.59831506,0.174532,0.92415595,0.81124026,0.8443375,0.21948099,0.9910173,0.013473883,0.6149283,0.027693966,0.5592991,0.84761155,0.8094379,0.30518973,0.008093831,0.50810325,0.6392387,0.8398242,0.2890962,0.6541464,0.4632378,0.92647725,0.33008948,0.2652367,0.38506648,0.46334,0.6549557,0.20831802,0.92330265,0.50718826,0.8800302,0.048143294,0.47973135,0.62129265,0.17242202,0.34416005,0.65868706,0.3305045,0.5157911,0.75957865,0.93819684,0.9893646,0.17303428,0.8170859,0.056698818,0.25782526,0.35345873,0.09125056,0.7210732,0.49699143,0.970733,0.0735816,0.5627797,0.07674086,0.93992794,0.86023825,0.54057795,0.1684083,0.5909065,0.16253005,0.011641049,0.35700518,0.7821705,0.63837844,0.12581013,0.87932825,0.9103641,0.62770915,0.61994696,0.19500826,0.28148246,0.9184461,0.5334663,0.722393,0.19540764,0.8032787,0.24683008,0.40349248,0.68704015,0.76823187,0.3352065,0.2816042,0.80858845,0.33747193,0.744123,0.6649029,0.67735904,0.03798913,0.39955038,0.6475329,0.5406856,0.5123951,0.23983105,0.3658833,0.9868024,0.12037513,0.21362446,0.36640835,0.40630093,0.511092,0.62350327,0.80283844,0.52577615,0.87910587,0.16484013,0.6506581,0.5693988,0.72696733,0.14864165,0.23753893,0.7503045,0.9398031,0.9816418,0.19725174,0.7923902,0.74302554,0.7790057,0.15850332,0.5168511,0.13226634,0.96873546,0.55794317,0.30060145,0.3822777,0.65768266,0.046340022,0.99187,0.86872953,0.010280746,0.7257704,0.98073095,0.62154245,0.1096403,0.6391436,0.56594133,0.73679084,0.4338758,0.4911501,0.9965093,0.097690746,0.71723795,0.5495111,0.7434422,0.76477414,0.060599875,0.632566,0.9735704,0.5160669,0.43873408,0.47247854,0.778155,0.6276335,0.9289001,0.43435636,0.75814146,0.23191643,0.11435921,0.5889161,0.6127571,0.4485074,0.8907834,0.73878855,0.98448014,0.7875962,0.48219055,0.108633436,0.933002,0.18155555,0.48917732,0.9648021,0.3498796,0.24523905,0.46179894,0.9286081,0.46839786,0.44040152,0.06512631,0.7179461,0.0046768077,0.8969666,0.85252285,0.077202,0.4642194,0.19511685,0.29715395,0.8886858,0.77644616,0.95990336,0.55000436,0.4204075,0.61194336,0.5744672,0.78577334,0.027326755,0.7502333,0.99585164,0.58549535,0.48871022,0.47126445,0.8136817,0.4787919,0.080777355,0.7197028,0.9160517,0.43460095,0.5458525,0.19786419,0.73828423,0.20805712,0.62396675,0.7486869,0.3176291,0.01362586,0.3704136,0.27712423,0.30617532,0.2797913,0.3744655,0.1216825,0.8539811,0.768459,0.9449136,0.96529144,0.74477166,0.9658291,0.17488709,0.13519672,0.12470057,0.056295514,0.55294275,0.1777468,0.16933882,0.7392772,0.53349274,0.20099632,0.7977428]
44bd3826-9511-411c-aa62-0b12e38584ed	S30	[0.8505896,0.31507555,0.61630166,0.37814546,0.043221075,0.70595104,0.20376095,0.696044,0.6702017,0.47677436,0.28651342,0.096462734,0.82645184,0.32524934,0.32604638,0.27278632,0.93290687,0.92520195,0.361525,0.38301468,0.80897456,0.78364134,0.32399204,0.2912045,0.76593983,0.39162466,0.66178864,0.55719346,0.95131606,0.7286853,0.1668561,0.7060677,0.7590612,0.061213337,0.42107442,0.7609737,0.67342293,0.8895601,0.26547435,0.9446459,0.9631191,0.6039855,0.21966241,0.54698884,0.009942132,0.87348026,0.91568905,0.07939601,0.70815766,0.93748456,0.8848137,0.5367098,0.8846447,0.4478205,0.5005321,0.368376,0.29885957,0.8373632,0.046640873,0.32090154,0.7343056,0.117517374,0.4134675,0.34759787,0.25311115,0.12056473,0.7969408,0.40774024,0.8175669,0.77514064,0.6811541,0.34430248,0.35500252,0.13249189,0.89289844,0.09560587,0.87343854,0.04070441,0.9507141,0.3331181,0.94701385,0.9342718,0.33712676,0.45119524,0.047013428,0.41549575,0.7935912,0.028035143,0.7267302,0.35058865,0.9678961,0.2113844,0.60081464,0.4851677,0.36023208,0.72778577,0.5256251,0.30410418,0.2699577,0.40690115,0.15223175,0.707469,0.8795535,0.47433016,0.06576489,0.19472775,0.7982209,0.86336523,0.30055627,0.3719677,0.45209005,0.9362436,0.30407354,0.057457156,0.31381652,0.033700608,0.5114641,0.29246038,0.4077612,0.2885979,0.5041926,0.12356755,0.44737425,0.8128597,0.9042777,0.3138525,0.92544967,0.9345667,0.06970425,0.96723217,0.27973133,0.14104323,0.98741955,0.4206133,0.8016069,0.84344065,0.80334276,0.2628726,0.7881587,0.23541154,0.14569713,0.42448547,0.96649355,0.80930424,0.5990446,0.22739953,0.00457656,0.99658203,0.4384225,0.7338109,0.27737218,0.37946704,0.9531594,0.985305,0.6982512,0.9936991,0.49652836,0.23138297,0.26929933,0.62961155,0.69042623,0.12705287,0.20622587,0.4772594,0.7549217,0.5851778,0.95535606,0.6321461,0.50920606,0.90173167,0.12422255,0.012737038,0.690255,0.92417777,0.21624853,0.20903748,0.6051966,0.34248707,0.03778148,0.42324907,0.08844032,0.020459851,0.7664059,0.0026383519,0.71065176,0.9381426,0.09759267,0.92436165,0.46063566,0.71767056,0.43033445,0.71736187,0.82916164,0.45665985,0.7450709,0.90100896,0.7870296,0.8293647,0.65411377,0.38026088,0.45028123,0.3156683,0.8703987,0.71403736,0.7971313,0.7161655,0.9552041,0.45871305,0.7001568,0.48030913,0.82838595,0.9143264,0.34441194,0.56467146,0.57115805,0.47068158,0.46570545,0.005256253,0.1853321,0.75978744,0.44994345,0.90303516,0.021832874,0.96731824,0.37135315,0.65607744,0.9862401,0.705682,0.8003216,0.56597227,0.15690282,0.5222275,0.17918354,0.42770356,0.28602314,0.9955335,0.9882963,0.16611505,0.2858658,0.85788864,0.0062416624,0.033951696,0.42734656,0.17458627,0.83603144,0.45411515,0.6399712,0.3691403,0.5271055,0.4105383,0.14857621,0.018436508,0.37157398,0.50928885,0.6130106,0.96012634,0.6024017,0.40819255,0.6377358,0.35013554,0.4231307,0.9278804,0.7654968,0.6495847,0.9384811,0.97332525,0.35709712,0.8414748,0.12663181,0.011494858,0.7584491,0.27206895,0.86849874,0.9639611,0.9497909,0.7705008,0.3177175,0.20750628,0.08819609,0.52585286,0.9002707,0.39157835,0.6378369,0.07057868,0.39207584,0.43862167,0.28909966,0.8989505,0.43673328,0.46669877,0.055826537,0.37997755,0.27779958,0.07647632,0.1803369,0.9160137,0.7737427,0.64448804,0.08571004,0.7733907,0.1631182,0.18783137,0.0702643,0.23790354,0.5274795,0.04498685,0.852687,0.5625616,0.66921943,0.561346,0.93641746,0.8644689,0.5965778,0.3051493,0.9326508,0.7442847,0.56529546,0.42044687,0.8063154,0.13187504,0.4824949,0.10816499,0.3105578,0.543698,0.19393308,0.058617275,0.71637845,0.29528227,0.036694113,0.43067223,0.0560279,0.0040197223,0.13562572,0.91805226,0.8849082,0.92702633,0.5067022,0.40861386,0.6283189,0.5336703,0.76381624,0.75816923,0.72219104,0.90921336,0.55862683,0.38450608,0.70454746,0.08639555,0.1683812,0.36362427,0.21027265,0.7097904,0.11755382,0.9323837,0.39413908,0.7908848,0.8813842,0.62422156,0.8311366,0.9023575,0.07822089,0.5033079,0.04232226,0.7213085,0.9193911,0.9549393,0.5846342,0.23157811,0.14147347,0.3525641,0.1377441,0.16839649,0.39542863,0.28245884,0.04987235,0.3008045,0.26351708,0.1394884,0.6403468,0.5478661,0.71314037,0.97790545,0.3081913,0.45162904,0.32705668,0.32222414,0.7864288,0.22052723,0.22378986,0.59434944,0.27372643,0.55380434,0.5581166,0.07493416,0.5915155,0.60297495,0.25210735,0.705501,0.45441803,0.8682328,0.7381913,0.09136039,0.6084176,0.8687167,0.09906617,0.68094534,0.29804432,0.31409773,0.695785,0.92478323,0.07565266,0.83726907,0.094435915,0.7477827,0.11086697,0.3259932,0.45237634,0.23770536,0.34060895,0.8442734,0.009011369,0.044765778,0.6110273,0.48340434,0.83868545,0.8327524,0.3435619,0.93300015,0.7858207,0.6600962,0.69468564,0.9300996,0.5353264,0.43582857,0.05467454,0.9333058,0.8446975,0.89794105,0.10428044,0.5004217,0.55436057,0.62136,0.14902653,0.49999848,0.52331793,0.7844617,0.07179059,0.011106463,0.73202676,0.31219116,0.16253842,0.03038275,0.34969997,0.5197398,0.22623542,0.033914853,0.5745258,0.99921936,0.90428233,0.28112766,0.93415356,0.7799576,0.7666912,0.5576744,0.26270154,0.7179879,0.7277809,0.03200079,0.45821077,0.96531487,0.8018858,0.51527095,0.23635533,0.53508246,0.34834018,0.80759794,0.9790135,0.98013216,0.91455907,0.77915114,0.1964267,0.6688759,0.8487307,0.5141356,0.70724916,0.94898033,0.6357107,0.77195954,0.76912934,0.6719605,0.30505598,0.90505004,0.06810846,0.32329887,0.63450885,0.7827733,0.80157846,0.3362482,0.05477611,0.81249034,0.13465133,0.38575703,0.5123909,0.7140649,0.098614216,0.5227706,0.9869039,0.08960128,0.005298412,0.100993015,0.26986217,0.004100676]
87cd50fe-5b68-4e30-9165-f4ae65303907	89532961	[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
f8455952-6f3f-485e-bd9b-942ce5bab472	S1	[0.26230365,0.637945,0.22045948,0.44492248,0.14878082,0.6247886,0.99441284,0.6931536,0.70531195,0.8028448,0.8003974,0.5887708,0.0033970533,0.7477663,0.4777026,0.5626351,0.008671098,0.20924707,0.47095177,0.083740436,0.6066703,0.7396059,0.99894416,0.93624485,0.2892692,0.31220964,0.7442396,0.53673726,0.2781413,0.93090236,0.2775284,0.42928374,0.88023114,0.9976746,0.43514127,0.32791367,0.15612148,0.9300959,0.5666193,0.2834875,0.95013106,0.49657986,0.4405865,0.3695472,0.46735594,0.81289387,0.21991092,0.02566114,0.6348483,0.6122518,0.63541174,0.9441789,0.13829432,0.8900373,0.34753188,0.17764513,0.75277644,0.46314177,0.11563626,0.6319785,0.3170159,0.21456699,0.50658447,0.6445064,0.023873644,0.26698944,0.12374122,0.110406816,0.06987097,0.11827455,0.74473864,0.108283855,0.5968138,0.49796307,0.97844815,0.84304786,0.589043,0.19562101,0.48200154,0.22621079,0.5918167,0.6752787,0.89527446,0.27656454,0.9824502,0.7501555,0.9868335,0.67876786,0.13353923,0.68718135,0.13120946,0.70872784,0.76403856,0.7816326,0.21627247,0.26526505,0.62225306,0.9488034,0.7944343,0.7378223,0.09123407,0.62561834,0.6105489,0.8093302,0.21949048,0.17366406,0.91285515,0.15643844,0.010258131,0.8313644,0.33831817,0.67468756,0.66576225,0.06385036,0.16964655,0.097229704,0.9210955,0.58169985,0.39336553,0.05869121,0.4206914,0.06660486,0.75155556,0.37711605,0.5746217,0.63397074,0.63666534,0.40197378,0.27841774,0.2917142,0.56679934,0.74573356,0.8895935,0.26590836,0.97170407,0.06996053,0.83002216,0.8457612,0.6565266,0.40517944,0.57344663,0.7694586,0.53758925,0.5176683,0.5599946,0.8679285,0.3774453,0.4230118,0.84759057,0.34441677,0.2677394,0.8835868,0.17441133,0.58243835,0.517976,0.32016596,0.07347745,0.31521457,0.07105711,0.46794096,0.4386175,0.95817214,0.75511986,0.92948097,0.33728075,0.048040263,0.02066718,0.34949848,0.89944017,0.2712995,0.6558191,0.83864284,0.23414089,0.68912894,0.9127149,0.6526432,0.08646539,0.1377021,0.91105473,0.06482394,0.09499113,0.30500984,0.30254182,0.19364764,0.8319693,0.09781659,0.12159133,0.8931977,0.68085194,0.23292956,0.8723206,0.8367126,0.46260086,0.48085442,0.019113628,0.60447097,0.6733345,0.050273463,0.7651433,0.29926327,0.66004944,0.9050167,0.6853718,0.5815457,0.9871209,0.7136202,0.12659015,0.046132427,0.30899358,0.19920126,0.4908599,0.12963386,0.54383796,0.62956715,0.5287167,0.9210306,0.12213953,0.7286122,0.45640263,0.08043453,0.08228973,0.5266275,0.11419213,0.76138467,0.24161045,0.7478958,0.45684084,0.032838758,0.8965909,0.97947574,0.7012655,0.42700946,0.5477554,0.63562506,0.16034234,0.46591792,0.6382263,0.9343752,0.23441191,0.26997295,0.34250587,0.85358566,0.7979348,0.44275573,0.5324739,0.158591,0.37523475,0.034786537,0.7768196,0.60231936,0.04750312,0.07368303,0.7572974,0.880728,0.34122652,0.85270464,0.77456266,0.90329295,0.8162143,0.34721336,0.72825205,0.10678977,0.6042572,0.03526664,0.9412178,0.38708574,0.92628485,0.7869093,0.34506768,0.18587483,0.79561126,0.13627121,0.8167386,0.035878446,0.39898938,0.1864119,0.25380766,0.71692944,0.0022033106,0.33696982,0.5834837,0.90648866,0.44519255,0.5025858,0.61465245,0.6623627,0.51142806,0.9807158,0.85029936,0.41597423,0.6702828,0.5809592,0.35459483,0.06949586,0.30763543,0.74681157,0.9282704,0.5953484,0.7794835,0.7997168,0.85358024,0.5128891,0.39200175,0.92084485,0.118868075,0.79029053,0.5275523,0.8321485,0.13037406,0.33307102,0.17918472,0.208045,0.9734027,0.88909805,0.818813,0.26371557,0.002390365,0.7581643,0.2127652,0.68591523,0.0071274205,0.3516462,0.23892097,0.22521758,0.11134849,0.92581934,0.24650683,0.74865776,0.60886616,0.45726907,0.92738336,0.07173759,0.22533247,0.30414099,0.01904897,0.60087043,0.66981536,0.4422795,0.4782613,0.9453136,0.74462926,0.75517356,0.13127197,0.11117796,0.070994586,0.5332828,0.07763159,0.38123813,0.88741636,0.46524256,0.68037766,0.69305015,0.5070888,0.38153708,0.8668393,0.25402197,0.07877879,0.46968687,0.80268556,0.4869793,0.020819396,0.16469102,0.8381201,0.87153137,0.32136422,0.040819176,0.24932006,0.70624673,0.42468897,0.1726163,0.23371236,0.9240153,0.052818496,0.20845819,0.15543297,0.14065963,0.1721625,0.6372368,0.08528216,0.6037989,0.99974555,0.24982719,0.068726175,0.58212537,0.11345005,0.1949614,0.2345381,0.91702664,0.6612306,0.029072868,0.08014282,0.87148726,0.4240265,0.16208193,0.99173224,0.2549029,0.05448923,0.8494238,0.21038103,0.8367723,0.8414825,0.8958816,0.04310569,0.5957365,0.17388329,0.18028921,0.10430689,0.84936607,0.4662675,0.96633834,0.48188648,0.74304444,0.36195183,0.4085834,0.9417254,0.31772265,0.7707352,0.76868874,0.3458996,0.49593318,0.9747956,0.90013874,0.20506231,0.60119903,0.010671091,0.7163582,0.37946644,0.5616011,0.94221735,0.6712762,0.69177306,0.166708,0.8048947,0.79938924,0.105739266,0.1661149,0.6089972,0.8472563,0.165321,0.9670091,0.9968082,0.8021479,0.09214024,0.19624738,0.082631655,0.95436627,0.34684935,0.880437,0.2943673,0.46617267,0.7769948,0.27028608,0.5899999,0.995407,0.9384064,0.3313791,0.31655464,0.24248822,0.556775,0.004377262,0.08292072,0.8498698,0.9753589,0.7339136,0.36321232,0.09287181,0.42173442,0.3836607,0.5088178,0.9822656,0.6865303,0.6532584,0.820239,0.2008385,0.99275196,0.03264129,0.5576426,0.44575164,0.48888156,0.36749253,0.2968073,0.23766425,0.31023324,0.88061833,0.6956229,0.33322763,0.8773958,0.19030982,0.9055561,0.15539773,0.40207475,0.2826981,0.5581942,0.44178313,0.5479276,0.43217528,0.11752704,0.57158494,0.41942304,0.032440204,0.37765437,0.45620018,0.48047835,0.052149385,0.21157578,0.4997247,0.7876538,0.10021038,0.2708565,0.5710156,0.41407168,0.4408931]
37a947b9-9a4b-4c99-908f-20aeb3010785	S3	[0.6168026,0.36525336,0.112555,0.04011891,0.26743162,0.45832735,0.9660476,0.99907565,0.7403818,0.11319198,0.92779106,0.052789755,0.23439184,0.19244206,0.035881713,0.9421789,0.07815648,0.08081244,0.41039765,0.97441,0.7105812,0.30449808,0.7210905,0.38271093,0.6062741,0.7850794,0.5387621,0.9887106,0.04468599,0.94328356,0.41312474,0.7281394,0.6034525,0.23836514,0.93546957,0.81683445,0.12771745,0.7929385,0.08524963,0.3853598,0.004438476,0.7038983,0.38481724,0.2398939,0.9030597,0.04823345,0.2640715,0.87271833,0.79086316,0.57399005,0.48075864,0.27562165,0.362224,0.02118435,0.45172682,0.8775048,0.63704646,0.79714125,0.64289993,0.6008509,0.5020046,0.40112698,0.38802376,0.25214615,0.7480777,0.035673928,0.9128166,0.7480719,0.051729944,0.6093329,0.043990154,0.7029012,0.99453896,0.78722847,0.10184648,0.78769815,0.41424477,0.1269539,0.3249973,0.74594283,0.9398501,0.18553352,0.29034874,0.47234613,0.28181916,0.81781775,0.4646036,0.08751285,0.16287634,0.20352116,0.28278813,0.9902376,0.37018794,0.7110987,0.07698833,0.336242,0.73953307,0.56154674,0.69745797,0.64821017,0.39233688,0.5484444,0.9896585,0.11197565,0.35739025,0.3642551,0.41178578,0.88471085,0.7693767,0.61008805,0.9492415,0.6349365,0.9118886,0.7397752,0.6678295,0.46448714,0.31951675,0.15746856,0.86601263,0.2118114,0.019860208,0.07925562,0.23237504,0.6112496,0.50555855,0.16003652,0.55318445,0.26586318,0.3456408,0.8527321,0.92137724,0.07868519,0.9454866,0.10815479,0.39174002,0.63614047,0.36851352,0.040249396,0.33187917,0.7016845,0.37920734,0.34091714,0.7150592,0.44926634,0.75135744,0.9357034,0.17779298,0.90486765,0.67522556,0.5386664,0.36806557,0.36927462,0.008873248,0.98408276,0.2093242,0.717863,0.3628963,0.66913956,0.48410803,0.49849123,0.078074,0.36154637,0.59590346,0.16501059,0.7041571,0.46595174,0.23213483,0.75318116,0.23889549,0.5690334,0.26216415,0.9574162,0.37164068,0.4727675,0.5143382,0.15430193,0.005655325,0.78654814,0.16345617,0.981271,0.36115807,0.36365294,0.045733705,0.9247128,0.7950616,0.6206206,0.040556747,0.4720407,0.6466652,0.82424164,0.7548212,0.2825647,0.11788783,0.9303732,0.30933037,0.8070659,0.19502077,0.43690714,0.74317515,0.3485233,0.90761703,0.258366,0.62666804,0.0051997514,0.5642219,0.62669003,0.44880185,0.7018473,0.6154184,0.06218367,0.06159442,0.49834123,0.87887615,0.42981863,0.8116205,0.9153414,0.073447384,0.24306723,0.7007545,0.62884843,0.75599647,0.6428461,0.008960174,0.92012125,0.7006703,0.88331056,0.6745805,0.4363186,0.439519,0.35043103,0.35875878,0.9031892,0.91306716,0.74041194,0.48695767,0.2689232,0.09661603,0.1434313,0.6356661,0.77106225,0.8030169,0.72113603,0.76296276,0.96755016,0.6715931,0.9651076,0.43707803,0.43041503,0.016740737,0.6656496,0.3640927,0.5464183,0.72756636,0.97051835,0.7064146,0.57295424,0.6235762,0.92660964,0.84526145,0.050254796,0.53879154,0.39451653,0.061051644,0.38904044,0.1562242,0.3775093,0.22066939,0.9137656,0.58066344,0.87875277,0.9829066,0.60675776,0.14637843,0.83273,0.042900126,0.5110393,0.7967695,0.23773889,0.11763558,0.9132948,0.0628488,0.8079065,0.035277408,0.6157598,0.16355753,0.2910859,0.66571814,0.002163126,0.964899,0.40968096,0.3948457,0.059629288,0.25218183,0.4801614,0.8954024,0.7396697,0.16711593,0.26817253,0.34195584,0.7093879,0.5004557,0.9604582,0.99916136,0.21955231,0.46901104,0.11026149,0.37169662,0.06500263,0.9964439,0.2910436,0.3470163,0.45861804,0.29105845,0.8336112,0.88968027,0.37552157,0.69978535,0.5551815,0.6923033,0.41660535,0.10029589,0.5543532,0.45253935,0.31601098,0.99782795,0.07619805,0.48345572,0.046895985,0.5232038,0.98957294,0.7607494,0.9636062,0.82652676,0.6791724,0.6560599,0.12476137,0.757068,0.4009141,0.34344366,0.8144228,0.6417223,0.65005547,0.22066462,0.51419175,0.89446044,0.71720403,0.09078159,0.09886379,0.7516038,0.18768841,0.55027765,0.7993997,0.21500397,0.74616706,0.5086961,0.9786527,0.6426495,0.53051245,0.5551178,0.8280982,0.03733681,0.72310954,0.30229416,0.46628073,0.20723669,0.6375749,0.6522227,0.429834,0.8964408,0.66701066,0.9691234,0.9497977,0.28214085,0.1720959,0.6581004,0.88605994,0.9064135,0.64466697,0.8317726,0.7920292,0.8586527,0.0016792482,0.62859875,0.6268639,0.6288288,0.21944825,0.04851148,0.49994186,0.2687639,0.24583232,0.99512756,0.5243143,0.44486937,0.778977,0.9217839,0.5319933,0.97319037,0.6617872,0.40467978,0.74902475,0.89131933,0.988586,0.7874137,0.5348944,0.063578,0.81455624,0.0034182793,0.4169816,0.43051276,0.20492175,0.082532406,0.41791946,0.87341964,0.4309106,0.039314292,0.51622784,0.38682336,0.67440027,0.25964108,0.8298603,0.6555467,0.08807271,0.77400446,0.8551239,0.06809773,0.11551359,0.5788903,0.055699803,0.038823355,0.085400365,0.5543433,0.8430028,0.84725314,0.40288123,0.49290404,0.6004083,0.4746975,0.35420963,0.18336658,0.7062497,0.2910415,0.23752889,0.27744836,0.13135234,0.80812716,0.7534157,0.7350707,0.046738736,0.8566445,0.9765088,0.08856046,0.24172229,0.025256759,0.3935344,0.2065936,0.67280275,0.22606382,0.4329752,0.05040192,0.9678384,0.2155806,0.25102168,0.5750503,0.46201748,0.93942034,0.4192368,0.3919645,0.3275835,0.95618504,0.36774167,0.4229341,0.5330871,0.43931675,0.3138742,0.028977644,0.3790579,0.47292888,0.41047037,0.5623112,0.019562714,0.5193762,0.94454694,0.29342678,0.23803534,0.8490728,0.40424806,0.12657714,0.71625865,0.4722013,0.1844421,0.8873271,0.010591683,0.5249416,0.6258497,0.93128824,0.7082569,0.24682692,0.92853934,0.49968588,0.79467905,0.5839802,0.040585306,0.5805103,0.19494785,0.6068869,0.93035394,0.18713811,0.39633593,0.80117714,0.51461947,0.19453442,0.36297]
2e927240-2f96-4da4-9210-bc92b9ce7b8c	S5	[0.11774144,0.9645719,0.60467327,0.8121509,0.020620985,0.63591546,0.2560825,0.4074342,0.009802432,0.15137854,0.95230865,0.5309314,0.033195306,0.9988037,0.01326852,0.5421912,0.49282765,0.6611417,0.15569118,0.03735428,0.5706054,0.10262109,0.2103936,0.36298558,0.43503132,0.06512923,0.46263072,0.4720939,0.4162008,0.4941957,0.6777428,0.9930719,0.53768295,0.08637427,0.40075997,0.2861588,0.8788544,0.98977256,0.7311211,0.2660839,0.53929824,0.9852921,0.58451235,0.79093707,0.7344267,0.999219,0.5508038,0.465392,0.30098614,0.04733679,0.22854216,0.07458731,0.5730871,0.4121525,0.6887668,0.43188947,0.8349554,0.09694733,0.22029632,0.13058631,0.031222455,0.7093267,0.05690097,0.6945595,0.30376118,0.18990481,0.76062936,0.18727158,0.08719715,0.3425728,0.3784926,0.8797548,0.6295177,0.87158763,0.3173699,0.46593282,0.2596639,0.19250071,0.21538144,0.10255276,0.4703333,0.113707185,0.34253168,0.98762715,0.3825255,0.2566816,0.44922456,0.23944287,0.84805894,0.06304503,0.96009326,0.9810125,0.81240815,0.93490213,0.3502182,0.698266,0.08727939,0.48733157,0.9194983,0.028456802,0.33432144,0.5452293,0.5689613,0.82563204,0.66513395,0.97721535,0.8892849,0.04346988,0.5669905,0.7546675,0.40525615,0.38258478,0.4202467,0.6877121,0.9383687,0.81467986,0.3902622,0.9765739,0.13120577,0.5849243,0.113323644,0.21953855,0.6319368,0.6406993,0.84858644,0.9440343,0.03439269,0.50686496,0.19564223,0.19706298,0.95904213,0.98800445,0.08062914,0.62859577,0.26496744,0.6322125,0.5280729,0.11608129,0.13799584,0.9741024,0.72821987,0.013664499,0.17334238,0.47694218,0.828443,0.2748236,0.43599966,0.017041968,0.4499115,0.06490762,0.540463,0.32738605,0.9423352,0.2819018,0.28089064,0.76241684,0.051730197,0.08811671,0.6913338,0.2688962,0.5927695,0.021511141,0.5359381,0.5626687,0.83887494,0.98360974,0.5144266,0.7376241,0.665158,0.8222778,0.052520055,0.5302298,0.17769729,0.8942247,0.1031547,0.06396821,0.8004004,0.9270824,0.325891,0.6057043,0.8966644,0.521019,0.9328337,0.7406911,0.85206676,0.9780839,0.8664096,0.6927387,0.79565126,0.31294626,0.13880354,0.31127295,0.11193601,0.42687815,0.7172998,0.5469895,0.2856368,0.6278133,0.65229654,0.48833746,0.18702695,0.2901671,0.89774954,0.82486075,0.64855444,0.58388114,0.66312784,0.16210474,0.7078952,0.51407033,0.775652,0.106262356,0.06523308,0.94502664,0.3531454,0.7066741,0.4199525,0.3951162,0.93132985,0.84649456,0.7804845,0.79176176,0.75388354,0.38126138,0.20546901,0.69714487,0.8140019,0.7394861,0.55848277,0.11524368,0.38792375,0.8078556,0.11503252,0.5475732,0.3949633,0.5888541,0.49340835,0.81715596,0.2817245,0.084162146,0.5244279,0.19832887,0.8268479,0.52183443,0.8924247,0.44864607,0.5671538,0.6896577,0.98104155,0.32643786,0.15965496,0.63399404,0.7727087,0.37164333,0.74656075,0.017314168,0.8818111,0.58504045,0.3500142,0.67607045,0.021586405,0.4387705,0.23803575,0.3761261,0.12599583,0.61351186,0.57947606,0.38777232,0.8836701,0.8715459,0.19514172,0.08627062,0.29203463,0.657088,0.88134825,0.020141056,0.58456933,0.5380031,0.18522732,0.16165915,0.55256885,0.4418295,0.554951,0.24819495,0.40498158,0.81484604,0.14287508,0.6528661,0.5648448,0.22764142,0.33470458,0.31464532,0.169265,0.46708956,0.26024738,0.079837754,0.6476109,0.86706716,0.28491715,0.2928113,0.7221058,0.75847787,0.82330805,0.8669972,0.94762945,0.38351458,0.24057247,0.09836739,0.5833115,0.83860797,0.4058499,0.28317353,0.9789026,0.068829894,0.58250356,0.9542313,0.44431472,0.032704916,0.24875677,0.69544536,0.94457746,0.5148106,0.5863456,0.69307256,0.86550504,0.8066513,0.42199776,0.96284175,0.8513034,0.46812505,0.594543,0.841244,0.26521084,0.66118693,0.4851117,0.13127649,0.0065928474,0.8219301,0.4510318,0.6049006,0.5815632,0.62981445,0.18109232,0.99824667,0.2319936,0.8226914,0.57334054,0.8143251,0.9725022,0.9698923,0.853089,0.36938578,0.93555886,0.5337177,0.6633865,0.020693239,0.11997475,0.93630815,0.95842123,0.80527663,0.4348307,0.96486974,0.14933367,0.21109764,0.14511633,0.899558,0.7271137,0.37892345,0.5459419,0.48469675,0.1338337,0.4830439,0.9820263,0.46101204,0.5447229,0.0066623483,0.35960984,0.32130533,0.8157039,0.584646,0.2745789,0.013174736,0.30643627,0.257797,0.0003666732,0.49833846,0.96879745,0.7362798,0.43782786,0.85025877,0.36634272,0.2493294,0.2972206,0.5882175,0.8834639,0.20467396,0.42093408,0.9485282,0.88085383,0.9611281,0.12592177,0.17457905,0.86730784,0.808018,0.8842638,0.4794316,0.5050323,0.7420744,0.94295555,0.31886593,0.8929053,0.7165105,0.9578574,0.77752924,0.23279418,0.29484084,0.75474054,0.4817296,0.8171147,0.21393119,0.0695,0.05369913,0.39043075,0.5023564,0.23017071,0.7111087,0.49278766,0.9535226,0.42322937,0.4146789,0.43771136,0.025280522,0.63537467,0.19244562,0.21287471,0.9665896,0.9285437,0.5196376,0.44335344,0.6298534,0.32345092,0.36935827,0.860308,0.75573474,0.5479652,0.3610575,0.1370329,0.71966124,0.6649207,0.39979693,0.74831784,0.37012783,0.8534287,0.645637,0.53223914,0.44385746,0.2618205,0.9560218,0.28531158,0.840903,0.93800753,0.13176014,0.9101878,0.38462138,0.74904454,0.1919938,0.60295236,0.94652975,0.76157176,0.8103047,0.52016324,0.16190448,0.4751199,0.047496196,0.8205107,0.3119891,0.87166756,0.27694318,0.81469727,0.89244235,0.60655624,0.8286387,0.8656739,0.19315213,0.033241306,0.8665235,0.6578618,0.20230152,0.84927154,0.9336818,0.03211714,0.23113267,0.9363302,0.972375,0.51900965,0.99770087,0.74784416,0.15169472,0.57753974,0.86606747,0.64346933,0.86871034,0.3537354,0.37408745,0.9408177,0.057295255,0.63133425,0.06539103,0.99327147,0.63623875,0.023820423,0.6879402]
874b8d9b-f901-4063-8b15-4305a35f708e	S6	[0.26922822,0.20200704,0.016153492,0.57517195,0.6954928,0.8410457,0.29237095,0.81320685,0.69767874,0.19128986,0.031533536,0.5551789,0.23355941,0.5902585,0.5520689,0.37105978,0.46765995,0.0013472395,0.0054149157,0.504389,0.18076593,0.50458026,0.7477709,0.7584022,0.3860849,0.26902276,0.41088387,0.22663447,0.2884285,0.80881125,0.4710452,0.99170625,0.011630609,0.85285026,0.057248313,0.257683,0.3896853,0.6679589,0.47076485,0.7713807,0.18175627,0.9481267,0.53747237,0.5844662,0.62532467,0.5056247,0.5573543,0.2308526,0.9240575,0.9373055,0.9328049,0.8295249,0.15185645,0.058970414,0.5257994,0.33555293,0.119432755,0.6789298,0.9342812,0.29391244,0.94047844,0.040276844,0.34490475,0.5048187,0.27531675,0.8837211,0.15078354,0.15540165,0.41448203,0.19896841,0.79898673,0.7502149,0.07581769,0.1547539,0.9283182,0.6969348,0.055223923,0.598941,0.47930527,0.11803472,0.9843314,0.8510881,0.70282257,0.96472156,0.16407374,0.03003623,0.41912007,0.8526465,0.44238552,0.13516589,0.6536428,0.04463975,0.9512334,0.40514222,0.25569502,0.8708125,0.941874,0.5290458,0.66576225,0.5392896,0.15358429,0.45065928,0.46009097,0.19936487,0.8519826,0.03756,0.3218738,0.3313707,0.20555277,0.9876613,0.38494834,0.15609227,0.19794169,0.3069539,0.2190372,0.42369246,0.96343565,0.9968076,0.49423248,0.798296,0.95656556,0.06135155,0.41982338,0.3459791,0.08099887,0.8870248,0.10856705,0.009788344,0.21613282,0.05617757,0.016207142,0.06152532,0.010403323,0.7286879,0.6549476,0.584154,0.41233617,0.12443152,0.691815,0.39288747,0.047148798,0.79934984,0.7414868,0.82260865,0.024644967,0.9550338,0.3225739,0.20367594,0.022253988,0.1060029,0.55541795,0.3947913,0.9971742,0.051648747,0.64816254,0.77112854,0.037320502,0.4769174,0.65977716,0.8069989,0.5694948,0.22744426,0.36277893,0.6700919,0.28935838,0.21409346,0.7049892,0.3931039,0.17119086,0.30011675,0.6395906,0.2928915,0.13717736,0.6746128,0.5844023,0.011869498,0.78833115,0.47269863,0.2736062,0.016944967,0.38470957,0.33472562,0.5912583,0.65302,0.020645522,0.13551778,0.5920582,0.32225993,0.91224706,0.9633508,0.6843137,0.10498369,0.76298714,0.6276345,0.8670934,0.6385569,0.47067288,0.3692771,0.9565224,0.89170617,0.02974023,0.81284827,0.035044633,0.41969958,0.26103634,0.6450875,0.30882603,0.67221254,0.400573,0.10511261,0.64794785,0.084920794,0.079566844,0.25486732,0.21177293,0.934632,0.23251331,0.70534766,0.96373945,0.5125431,0.35434455,0.3140549,0.9655605,0.8350464,0.26646245,0.3338898,0.063919105,0.5907506,0.24036007,0.18052648,0.8168473,0.3802723,0.058229882,0.19396235,0.45144507,0.8214518,0.14214277,0.19739075,0.96868426,0.29051986,0.42117575,0.07602115,0.13966756,0.25560167,0.8700642,0.47656855,0.3984944,0.73597425,0.19477679,0.95498604,0.6638497,0.2857776,0.23895988,0.01842228,0.7812766,0.86551917,0.25417936,0.7079074,0.24914359,0.5086121,0.73147416,0.13417327,0.5835734,0.20739098,0.52954173,0.666045,0.062748305,0.3603344,0.7039541,0.9886472,0.4229546,0.47935826,0.783332,0.0009918337,0.7452406,0.29862943,0.630125,0.06780557,0.6629879,0.97057885,0.60051095,0.9322301,0.69898385,0.21023788,0.2630667,0.49624038,0.98399174,0.56579614,0.12857999,0.3925024,0.87853384,0.27878433,0.176571,0.7911239,0.32829016,0.69975615,0.90598774,0.12967485,0.7781541,0.3237532,0.6973643,0.51598257,0.34975383,0.2092044,0.03885565,0.072488345,0.09971197,0.54414815,0.45749274,0.012646387,0.2973991,0.6169357,0.02395271,0.19207653,0.7873299,0.73660517,0.939912,0.72922504,0.28593642,0.11019336,0.67853415,0.73502797,0.98037004,0.8143067,0.7000414,0.35543424,0.028834585,0.7073297,0.876881,0.12292668,0.14245126,0.1469507,0.37383664,0.83357906,0.45489714,0.047913782,0.52390844,0.8489159,0.14061557,0.7592303,0.20908454,0.08046432,0.34963465,0.75001687,0.14524636,0.7053821,0.13261516,0.7556071,0.86252594,0.885882,0.95824957,0.09965107,0.18471768,0.32663673,0.16328795,0.32094756,0.71216756,0.76388085,0.30352738,0.21870954,0.792125,0.10416567,0.43692434,0.4093677,0.63018,0.7835562,0.12300037,0.03848682,0.078880884,0.8298177,0.41454637,0.5199082,0.55315787,0.6064037,0.053401038,0.27972582,0.43286481,0.35360157,0.9520066,0.6710957,0.71143556,0.0214137,0.85789305,0.4197583,0.25045726,0.8611253,0.5990573,0.094563946,0.7986323,0.93424225,0.44390565,0.7147967,0.15640074,0.098816924,0.54831576,0.20045026,0.50194645,0.31987798,0.7865661,0.809449,0.7966552,0.6572912,0.20083301,0.6137319,0.100003704,0.6665714,0.5753006,0.55907416,0.3005853,0.42077962,0.6280088,0.26914927,0.4818744,0.3576774,0.00033576318,0.12351257,0.8868259,0.030092485,0.62851214,0.99441385,0.8732902,0.576994,0.0034998443,0.40847966,0.017637113,0.0015821414,0.95080286,0.40240043,0.24227846,0.22264811,0.4708604,0.43674418,0.37717682,0.9253802,0.13506871,0.63286793,0.7282578,0.7184606,0.2738296,0.116081975,0.70966595,0.5364621,0.79830724,0.43671918,0.24451467,0.8005466,0.13098702,0.73054135,0.090932794,0.67881364,0.80983925,0.92993885,0.040471975,0.18874183,0.60209286,0.079732284,0.3881678,0.061126508,0.7668762,0.14484666,0.035837613,0.37536117,0.78292394,0.76704407,0.9464511,0.8250102,0.43254325,0.1796201,0.2456028,0.48669872,0.7564236,0.38059774,0.110577874,0.74431616,0.9491009,0.7860703,0.59162354,0.9068978,0.7652776,0.8329326,0.88993204,0.105028905,0.6275634,0.3141358,0.9025096,0.5230947,0.8491829,0.20181946,0.025012316,0.46723923,0.36926028,0.21471046,0.3983349,0.9658158,0.13603234,0.53769505,0.06419565,0.3087571,0.5674275,0.75576437,0.8345803,0.9766612,0.22218482,0.40396672,0.22971007,0.25522253,0.11642512,0.98673886,0.78599316,0.84443957,0.8256934,0.84052587]
0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	2115239	[0.045842,-0.0078486,-0.040123,0.0075447,0.0010207,-0.00036376,0.0026668,0.0080063,-0.014787,-0.035615,-0.058673,0.013496,0.046239,0.0083524,0.037935,-0.030408,-0.001726,0.029764,0.006373,0.036317,-0.022101,0.043667,-0.086445,0.026312,-0.0079926,-0.099313,0.05039,-0.023234,-0.011128,-0.042013,0.062553,0.011786,-0.023039,0.029377,0.011221,0.073149,0.020404,-0.002704,0.067862,0.034153,-0.0034736,-0.045633,0.031721,0.03212,0.034079,0.018895,0.058275,-0.021068,-0.0069417,-0.049316,-0.0043892,-0.020393,0.022048,0.048216,0.033087,-0.066251,0.069351,0.0037191,-0.043628,-0.066624,-0.018504,-0.13775,-0.013842,0.004976,-0.04108,0.051396,0.088645,-0.024716,0.0009478,0.076273,-0.045006,0.018402,0.027696,-0.038654,-0.014228,0.0055964,0.028569,0.051161,-0.026527,0.054628,-0.0047583,-0.065713,0.027278,-0.027126,0.036026,-0.012166,0.01882,0.026029,0.034416,0.0053407,-0.028941,0.027593,0.011909,0.047959,0.028839,-0.044521,-0.043786,0.022104,0.011645,-0.054536,-0.030069,-0.068628,-0.025344,0.062275,-0.032923,0.036909,-0.012944,-0.022874,-0.090989,0.051047,0.035849,-0.00010332,0.10183,0.0302,-0.025383,0.00040719,0.02327,-0.0051338,-0.078379,-0.0024296,0.011727,0.025532,0.011328,0.051316,-0.030736,0.019931,0.0044604,0.00037281,-0.02401,-0.0011501,-0.061827,0.015571,0.057307,0.018557,-0.030071,0.036245,0.032075,0.016153,0.0046587,-0.016177,-0.079189,-0.066471,0.025924,-0.075903,0.02593,0.069944,-0.011042,0.026051,-0.0049498,0.040081,-0.023873,-0.025872,0.011445,-0.0016935,-0.0092523,-0.019389,0.020622,0.07401,-0.057622,-0.045737,0.040377,-0.080436,0.052074,-0.038648,-0.010464,-0.04387,-0.038035,0.060298,-0.0037682,0.0435,0.065231,0.038323,0.045121,-0.024573,-0.0098254,0.017139,0.0092533,-0.069644,0.086735,0.0019462,0.019699,0.095201,0.0080513,0.012755,-0.024496,0.018983,-0.0009362,0.046655,0.0010492,-0.033156,-0.013834,-0.031615,0.034571,-0.0092786,-0.053384,-0.022881,0.027582,0.0065483,0.017343,0.032426,-0.087095,-0.023541,-0.043295,-0.031249,0.024466,0.050219,-0.050389,0.033488,-0.027861,0.083732,-0.04173,0.029674,-0.010674,-0.052115,0.04903,0.021277,-0.050479,0.01213,-0.0097588,0.08218,-0.031994,-0.0027291,0.085239,-0.016557,0.059583,0.11523,-0.010787,0.061625,0.067282,-0.027763,-0.026852,0.041826,0.018394,-0.020209,-0.096908,-0.016628,0.0083812,0.033416,0.0479,0.062208,0.027506,0.028421,-0.011568,0.048786,-0.024615,-0.087665,0.015741,-0.018104,-0.0040916,-0.039808,0.031955,0.031181,-0.027011,0.03819,0.0021132,-0.029736,0.049647,0.07047,0.010693,-0.0056387,-0.0042572,-0.044562,0.014838,-0.0029421,-0.046823,-0.0047644,0.023081,-0.019094,-0.014811,-0.036373,-0.056202,0.025327,0.033905,-0.026402,0.02552,0.017126,-0.0093368,-0.044654,-0.052924,-0.024716,-0.011734,-0.0024479,-0.067616,-0.016278,-0.010419,-0.041882,0.093947,-0.022765,0.031917,0.0056335,0.11905,-0.061939,0.053311,-0.021416,-0.046165,-0.0076536,-0.0056303,-0.070221,-0.0092919,0.014764,-0.020238,0.021639,-0.0070229,0.020204,0.039805,-0.0072238,0.023654,-0.00047169,-0.042548,-0.0058126,0.04434,0.0038856,0.018228,-0.0077585,-0.015172,-0.040441,-0.041495,-0.025094,-0.003109,-0.060817,-0.0053584,0.027309,0.024433,-0.023014,0.0085574,0.029705,-0.063289,0.014192,0.031164,0.026658,-0.044774,-0.013922,-0.023613,0.12959,-0.028433,0.063816,0.068158,0.030578,0.0020169,0.0097171,0.0096626,-0.092518,-0.067888,-0.059778,-0.043247,0.013003,0.032773,0.10876,-0.031093,0.034715,0.033932,-0.01081,-0.013401,-0.0041842,0.11638,-0.044314,0.077119,-0.031051,-0.037828,0.0066502,-0.069358,0.035737,-0.0056944,0.072049,0.018759,0.011266,0.13277,-0.085395,-0.016939,-0.022716,0.047337,-0.089872,0.0029566,0.015082,0.024792,-0.00012064,-0.018123,0.058066,-0.0054611,-0.014086,0.038109,0.060426,-0.058849,0.0067548,-0.023732,-0.0087081,-0.0651,-0.020514,-0.023134,-0.029152,0.010652,0.067383,-0.066789,-0.076673,-0.0037118,0.075836,0.051906,-0.0021955,-0.061521,0.03898,-0.027008,-0.016878,0.034082,0.0022509,0.0080772,0.0073148,-0.0014052,0.00018868,-0.0071001,0.088971,0.018844,0.066553,0.060927,0.0097067,-0.042611,0.023483,-0.046577,0.13133,0.0078296,-0.017042,-0.025606,0.0056241,0.0073918,0.020395,0.020271,-0.043811,0.0062495,0.077921,0.022327,-0.0032256,-0.021721,0.067928,0.0085471,-0.0038906,0.048342,0.031317,0.0053095,0.031486,0.096881,-0.061517,-0.01081,0.021196,0.13792,-0.057368,0.012242,-0.066725,-0.048699,0.15152,-0.0064172,0.030474,0.053268,-0.0075716,0.018491,0.0098103,0.013559,0.060047,0.094276,0.047934,-0.070002,-0.0038576,0.016083,0.023367,-0.039205,-0.037597,0.0039369,-0.012973,-0.020828,0.06405,-0.043179,-0.011674,-0.10233,-0.013761,-0.040444,-0.03523,0.038324,-0.010057,-0.055276,0.11373,0.077798,-0.03254,0.11359,0.051966,0.041278,-0.046385,0.11399,-0.023813,0.021976,-0.038244,0.050125,-0.060507,-0.050174,-0.027862,0.031667,0.0044948,-0.031987,0.029021,0.055567,0.065169,-0.0021198,-0.0069689,0.026177,-0.036289,-0.033029,0.072066,-0.00034197,0.08478,0.01276,-0.014038,-0.036885,0.0070811,0.019452,0.034402]
286c9234-c533-4ef6-8774-a5de92fb8452	2115253	[-0.030868,-0.0044425,-0.037824,-0.014401,-0.0026246,0.029267,0.036208,0.021268,0.043891,0.0023229,-0.033856,0.049058,-0.032508,-0.021382,0.059022,-0.018429,-0.1074,0.097261,-0.056799,0.054145,-0.052758,-0.013724,-0.063136,-0.027642,0.031376,-0.036685,0.0079783,-0.028246,0.019337,-0.040601,0.0090831,0.045117,-0.045458,0.062501,0.048398,0.031106,0.077065,0.049366,0.024639,0.056521,0.0079612,-0.0059939,0.017501,-0.0042863,0.056027,0.028017,0.018359,0.0026323,0.0014714,-0.033157,0.027516,-0.054825,0.023154,0.032581,0.034718,-0.051373,-0.034129,0.064083,-0.0074642,-0.0013057,-0.027277,-0.085306,0.038505,-0.051668,-0.027299,0.10631,-0.096627,-0.019452,-0.075189,0.12637,-0.012321,0.027246,0.05487,0.065723,0.053022,-0.036514,-0.016146,0.022558,0.0043302,-0.0029706,-0.047062,-0.054388,-0.026871,0.011883,-0.015039,0.036251,-0.038592,-0.024249,-0.0073797,0.022092,-0.018787,-0.068844,-0.010425,-0.083299,0.0099112,-0.01527,-0.017403,-0.0023075,-0.042884,0.072585,-0.026973,-0.082972,0.046376,0.048964,-0.041165,0.023111,-0.01972,0.014286,-0.048558,0.098166,0.0481,0.0020423,0.063449,0.027328,0.0077329,-0.0043477,0.015323,-0.042261,0.009518,0.00077537,-0.028259,0.053885,0.064832,-0.04556,0.010472,0.036452,0.0029756,0.032111,0.027635,0.003298,0.036404,0.081728,0.028215,0.097023,-0.03168,0.016737,-0.032037,0.024959,0.051088,-0.0071356,-0.067079,-0.045372,0.040944,-0.058284,-0.0038945,0.025129,-0.042119,0.023442,0.036396,-0.013883,0.057939,-0.013046,-0.025271,0.032692,0.0080788,-0.041652,0.010039,0.043157,0.024808,-0.037189,-0.0038448,-0.051059,0.10427,-0.042775,0.036102,0.00054263,0.0045365,0.016048,-0.0023158,0.03909,0.033784,-0.0053354,0.0054179,-0.0094271,-0.022577,-0.0023587,-0.034024,-0.072618,0.081137,-0.042989,0.023868,-0.058856,-0.034022,0.00066549,-0.066252,0.0051894,-0.0031573,-0.0006662,-0.024309,-0.033398,-0.044584,-0.043878,-0.02218,0.021262,0.050046,0.045156,0.068841,-0.053371,0.022278,0.099475,-0.01754,0.0081045,0.0055915,-0.00039539,0.03645,0.030556,-0.021977,0.035063,-0.050472,-0.014591,-0.088343,0.02682,0.042466,0.023538,0.047554,0.080467,-0.08593,0.061067,0.014438,-0.026314,-0.037209,-0.01718,0.043768,0.027876,0.025989,0.0068382,0.071649,0.010586,0.033072,-0.0056919,-0.080801,-0.0056036,-0.023173,-0.0049065,-0.047665,-0.0090869,0.0042835,-0.02697,-0.0042465,0.084452,0.073839,0.042307,-0.039779,0.00095478,-0.058348,-0.061056,-0.032807,-0.061732,0.020686,-0.057111,-0.06279,0.042311,0.0039196,0.041725,0.016938,-0.012846,0.036198,0.00062917,-0.037457,-0.011004,-0.051443,-0.014461,-0.012582,-0.025767,0.012079,-0.0039632,-0.051081,-0.0076636,0.034394,-0.0073989,0.03052,-0.0012762,-0.086147,0.018603,0.028459,0.014203,-0.036694,-0.057321,0.0093023,-0.096168,-0.047457,-0.10454,-0.03979,-0.012702,-0.099236,0.024361,0.057795,0.014104,0.049048,-0.0048747,0.051162,-0.018143,0.022483,-0.086065,-0.031855,-0.049233,0.012081,-0.070561,0.0082292,0.035639,-0.045373,0.030572,-0.041871,0.018483,0.031564,0.028326,0.045144,-0.0054044,0.0050203,0.041522,0.095621,0.012046,0.0065057,0.022006,-0.012824,0.042124,0.034473,-0.0056746,-0.059264,0.020325,0.038467,-0.072681,0.046551,-0.0088855,0.0041197,0.039296,-0.066122,-0.036224,-0.0013221,-0.016229,-0.034717,0.094301,-0.029927,0.013417,-0.046973,-0.020848,-0.039982,-0.050024,0.040236,-0.015347,0.059618,-0.051215,0.0032198,-0.095775,-0.029263,-0.039814,0.050349,0.023706,-0.098223,0.057045,-0.00033654,-0.082374,0.0074401,0.031408,0.038132,-0.045645,0.015983,-0.046219,-0.045095,0.10926,-0.052053,0.047734,0.0086152,0.038158,0.019868,0.033354,0.099676,0.019939,-0.010965,0.024476,0.050945,-0.013971,0.019129,0.059659,-0.036013,0.060074,0.039544,0.053569,-0.064764,0.00029856,0.030216,0.058027,0.038968,-0.027384,-0.037741,-0.0072843,0.023053,-0.11991,0.04372,-0.016989,0.012728,0.053061,0.030179,-0.044112,-0.0059288,0.035312,0.052422,0.053916,-0.0025529,-0.027902,-0.066507,0.098485,-0.016201,-0.015664,-0.060118,0.016982,0.019263,-0.056992,0.025916,0.055009,-0.015498,0.057849,0.066134,1.7767e-05,-0.055363,0.019994,0.015465,0.089184,0.013069,-0.022198,0.036208,0.0059437,0.029668,-0.03482,0.0024706,-0.062384,0.029945,0.037885,0.07179,-0.0097524,0.011281,0.018312,-0.03306,-0.0036147,-0.044038,0.13999,0.013974,0.031428,0.08139,0.016643,-0.060578,0.026748,0.060264,0.033679,0.004519,0.00096547,-0.046661,0.10594,0.017114,0.0029836,0.072526,-0.022513,-0.047603,0.055067,0.0012137,0.058119,0.010519,0.10217,0.034833,-0.025345,-0.058779,0.0082387,-0.039365,0.0073973,-0.023369,0.0094109,-0.010407,0.047662,0.05321,0.0080495,-0.02163,-0.003523,-0.10366,-0.042701,-0.041369,-0.027327,-0.062202,-0.0074319,0.023661,-0.017611,0.046583,0.046409,-0.017788,0.033984,0.022917,0.032042,-0.013659,-0.018154,0.07751,-0.050428,0.026611,0.039142,0.074253,-0.051368,-0.078751,0.042534,0.058962,-0.036621,0.025952,0.090299,0.0004217,-0.053991,-0.0281,0.050976,0.011473,0.047866,-0.0031929,0.010486,0.0062352,0.039937,0.069185,-0.00099146]
0ff33482-fc8b-4a12-b6e8-5ed58070d21e	2111881	[0.056,0.015,0.0066,0.0513,0.0058,-0.0034,0.0312,0.0731,0.0081,0.0514,-0.0964,0.0923,0.0694,-0.0169,-0.0286,-0.0014,-0.0189,0.0248,-0.0129,0.061,0.0209,-0.049,-0.0179,-0.0774,-0.0424,-0.0366,0.0281,-0.0142,0.0009,-0.0117,-0.0056,-0.0018,0.0231,-0.044,0.0508,-0.0287,0.1081,0.106,0.0071,0.0482,-0.014,-0.029,0.0215,-0.0222,0.0731,-0.0064,-0.0073,-0.02,0.0375,0.0044,0.0004,0.0273,-0.0037,0.0141,0.0186,-0.0684,-0.0631,0.038,-0.0143,-0.0017,0.0257,-0.0901,-0.0081,0.0562,0.0554,0.0777,-0.0589,-0.0234,-0.0023,-0.0682,0.0956,0.0348,0.0878,0.1208,0.02,0.0113,-0.0355,0.0605,0.0204,-0.0449,-0.0995,-0.0473,-0.0354,0.02,0.0266,-0.0496,-0.0024,0.0926,0.0129,-0.0084,-0.0392,0.0765,-0.0192,-0.0125,0.0231,0.008,-0.0161,-0.0851,-0.0363,-0.0052,0.0121,-0.022,-0.03,0.0164,-0.0286,0.0182,-0.0257,-0.0411,-0.0652,-0.0289,0.0704,0.0382,0.0777,0.0249,0.0774,-0.0339,-0.0263,0.0656,0.0103,0.0322,-0.0169,-0.0147,-0.0087,-0.0449,0.0489,0.0488,0.0136,0.0291,-0.0753,0.0055,-0.0182,0.0611,-0.0578,0.1246,0.0013,0.0326,-0.0436,-0.0737,0.0318,-0.0319,0.0415,-0.0474,-0.0134,-0.0198,0.0493,0.0026,-0.0351,0.0229,0.0155,0.0137,-0.0121,0.0254,0.0432,0.0291,-0.0192,0.0263,0.0149,-0.0487,-0.0052,-0.017,-0.0507,-0.0049,-0.0309,-0.038,-0.0114,-0.0118,-0.0351,0.0172,0.0084,-0.0164,-0.0321,0.0197,0.129,0.0188,0.0445,0.0096,0.0031,-0.0062,0.0175,0.039,-0.005,0.0048,-0.0695,-0.0234,0.0326,-0.0065,-0.06,-0.0322,-0.0286,0.0383,0.0014,-0.0938,-0.0272,-0.0362,0.0521,-0.033,0.0117,0.0355,0.0158,0.0921,-0.0884,0.0375,-0.0027,0.0343,0.0705,-0.0226,0.0261,-0.0062,0.0218,0.0169,-0.0489,0.014,-0.0032,0.0104,-0.0082,0.0049,-0.0064,0.0183,-0.0361,0.0657,-0.011,-0.0039,0.1292,-0.0669,0.0473,-0.0819,-0.0457,-0.0407,0.0182,-0.0037,0.0039,0.0115,0.0931,-0.0019,-0.0332,-0.0728,0.0254,-0.118,0.0491,0.0441,0.0166,-0.0146,0.0093,0.0174,-0.0339,-0.0698,0.0009,-0.0348,0.0007,-0.0615,0.0282,0.068,0.101,-0.0998,-0.0101,-0.0445,0.0105,0.0346,0.0373,-0.0407,0.0172,0.0311,-0.0045,-0.0605,0.005,-0.0212,0.0012,0.0633,0.0592,-0.0146,-0.1238,0.0393,-0.0936,-0.002,0.0081,0.0314,0.0038,-0.0083,-0.0345,-0.0139,-0.07,-0.022,-0.0836,-0.0295,-0.0059,0.0147,0.1031,-0.009,-0.0213,0.0004,0.0645,-0.031,-0.0585,-0.0828,0.0742,-0.0521,0.0486,-0.0248,0.0487,-0.0108,-0.0182,-0.035,-0.0519,-0.0283,-0.0769,-0.0181,0.0493,-0.015,-0.072,0.0545,0.0146,-0.0308,-0.0026,0.0452,-0.0252,0.0124,-0.0824,-0.005,-0.0021,0.0157,0.0168,-0.0572,0.0744,-0.0272,-0.0337,0.0639,-0.0131,0.0065,0.0527,0.0587,-0.006,0.0049,-0.0842,0.0208,-0.0451,0.036,-0.0013,0.0073,0.0717,-0.0019,-0.0362,-0.0313,-0.0211,-0.0216,0.009,-0.0278,0.0249,0.0341,0.0864,0.0211,-0.0012,-0.0217,-0.0089,-0.0411,-0.0212,-0.0607,-0.0114,0.0359,0.0133,0.1144,0.0069,-0.0251,-0.0033,0.0142,-0.031,0.0391,0.0608,0.0242,-0.0088,-0.018,-0.0062,0.0161,0.0071,0.0278,-0.0042,-0.0291,-0.0323,0.114,0.0012,-0.0172,0.0029,0.0692,-0.0723,0.0588,-0.0446,0.0133,0.0496,0.0325,-0.0589,0.0002,0.0839,-0.0104,0.0296,-0.0203,0.0062,0.0607,-0.0403,0.0078,-0.0062,-0.0026,0.0005,0.0758,0.0727,0.037,-0.0322,0.0491,0.0086,-0.0602,-0.0295,0.036,-0.0064,0.0495,0.0168,0.0311,-0.0227,-0.0244,-0.0233,0.0507,0.0486,-0.0342,0.0193,-0.0241,-0.0078,-0.0053,0.0065,-0.0139,-0.0215,0.0528,-0.0339,0.0462,-0.0058,-0.0149,-0.006,-0.0177,0.0422,0.0538,0.0583,-0.0617,0.111,0.0708,0.0231,-0.0644,0.0493,0.0026,-0.0647,-0.0388,0.0578,0.1086,0.0162,0.0417,0.0262,-0.0039,0.0139,-0.0026,0.0008,-0.0086,0.0153,-0.0607,0.009,0.0267,-0.0326,-0.0786,0.0176,0.0325,-0.0145,0.0041,0.0138,-0.0029,0.0124,-0.0079,-0.1153,0.0801,-0.0013,0.0066,-0.0253,-0.0218,-0.023,0.0088,0.0111,0.0122,0.0681,0.0407,0.0755,-0.0439,-0.0182,0.0196,-0.072,0.0186,0.0814,-0.0495,-0.0501,-0.0144,0.0272,-0.0169,0.0514,0.0342,0.0133,-0.0294,0.0222,0.0343,-0.0303,-0.0242,0.0749,-0.0121,0.0462,0.021,0.0746,0.0192,0.0915,-0.011,-0.0346,0.079]
3ef8aead-9239-43dc-9a7a-d9b3df374a99	S9	[0.81643087,0.67910415,0.5474869,0.6559948,0.31169006,0.7463474,0.7074565,0.53901875,0.43144313,0.5113036,0.5066886,0.9033868,0.22695298,0.3445089,0.8890483,0.8413243,0.5944313,0.059146453,0.9013197,0.56122565,0.9842985,0.9723026,0.91144574,0.22259058,0.33904397,0.9248855,0.59647775,0.5757395,0.86618936,0.923004,0.07487403,0.0006687784,0.9705928,0.42997536,0.9885195,0.23030663,0.7846222,0.69102705,0.3447381,0.35369414,0.23433125,0.63158065,0.9196609,0.72094417,0.55530006,0.06770647,0.62390953,0.5681218,0.3282772,0.32203552,0.69962883,0.9893019,0.25528145,0.9926189,0.27075976,0.5672947,0.806112,0.1198373,0.7074766,0.5729384,0.3595249,0.020399036,0.44313267,0.028173797,0.31041026,0.49898055,0.21398859,0.857033,0.46340108,0.6174749,0.10780455,0.6413299,0.99135476,0.14324032,0.7564499,0.90834266,0.2962197,0.33374372,0.31617838,0.11602613,0.089985654,0.4610792,0.6387853,0.013455019,0.18287994,0.6027545,0.36309287,0.4725729,0.5993136,0.15868725,0.47749,0.23224157,0.3667452,0.25262675,0.4567471,0.98106635,0.9650203,0.006307818,0.43354595,0.856191,0.65127003,0.99435115,0.53908706,0.74162775,0.057630546,0.35266906,0.07976955,0.15848412,0.5969515,0.7459545,0.8857484,0.18352039,0.57115,0.16332884,0.3527274,0.9937224,0.1011623,0.8039222,0.7233471,0.75907046,0.5315096,0.118820235,0.26470247,0.34771517,0.76682776,0.042482503,0.59026706,0.8335632,0.53536415,0.44930524,0.81248546,0.74693954,0.36886257,0.68327254,0.44679266,0.73048884,0.63439727,0.9572906,0.7941131,0.15158057,0.56095743,0.524061,0.5793698,0.9076956,0.4627331,0.21109551,0.18326974,0.4914796,0.9795431,0.37776497,0.32575122,0.9351238,0.88264865,0.27965537,0.69237745,0.2773382,0.4983882,0.26803657,0.58463824,0.15592125,0.44859767,0.52373236,0.7317413,0.3085969,0.92315525,0.07488384,0.77561426,0.9259613,0.8629052,0.008957845,0.09268359,0.71268266,0.5640366,0.41827407,0.5150206,0.22922324,0.9937652,0.9781772,0.28706124,0.3122438,0.35701805,0.9619792,0.6669462,0.14223671,0.0010903649,0.39034724,0.8646997,0.7882068,0.9196545,0.55284315,0.9382172,0.2504654,0.88157344,0.7479899,0.4841586,0.39975306,0.10756924,0.79607177,0.6107038,0.35149255,0.53349763,0.81325155,0.405603,0.60094434,0.05220061,0.3144981,0.18713935,0.22366677,0.8075837,0.24314676,0.6031137,0.13576098,0.9464073,0.5584643,0.6658324,0.4095126,0.37106276,0.7077358,0.5645446,0.0110214045,0.82008773,0.051240418,0.75523216,0.559451,0.28636128,0.92761517,0.07993167,0.6280954,0.636769,0.106852494,0.0042904126,0.6860015,0.07506242,0.6315458,0.7501688,0.98009425,0.040977474,0.86963713,0.14293458,0.5355521,0.41271123,0.024898255,0.98779863,0.96523315,0.41276863,0.09714223,0.14955822,0.6744063,0.17309351,0.30277142,0.009640284,0.7876153,0.36982232,0.8573643,0.66790193,0.26477444,0.66194946,0.6116532,0.43356875,0.89032537,0.95299107,0.2880783,0.72939026,0.4553489,0.7009166,0.9547232,0.12715666,0.7690966,0.3204908,0.34817928,0.2950988,0.1621917,0.660332,0.91599303,0.19485134,0.28218243,0.30999175,0.9997515,0.992104,0.9496877,0.9380995,0.91341025,0.03984011,0.10770701,0.4341292,0.8074406,0.33798006,0.93389,0.3195697,0.6238911,0.18576877,0.51258314,0.63941693,0.8444729,0.55628693,0.93550897,0.6876214,0.8717807,0.23230349,0.6273214,0.41709787,0.14553836,0.9567377,0.8732577,0.83271235,0.32551426,0.3182043,0.5991236,0.9204015,0.4723784,0.07052278,0.11972267,0.7089769,0.30102298,0.047188204,0.7387717,0.31329533,0.5058023,0.84591764,0.5765355,0.64045686,0.32353854,0.05421738,0.18963042,0.15852547,0.59383774,0.79361045,0.2830234,0.8151761,0.6132646,0.9746894,0.80676085,0.21134144,0.59545374,0.7702132,0.061994977,0.34215128,0.441727,0.8172497,0.9680412,0.2534439,0.6667198,0.3453547,0.44659898,0.747919,0.6389791,0.38129497,0.3363151,0.81981176,0.28645962,0.7695398,0.18072435,0.6016114,0.2500649,0.8845516,0.078002565,0.14246877,0.20376824,0.6348699,0.69860417,0.19278534,0.20146193,0.89162046,0.72930276,0.9941669,0.7279054,0.7342522,0.8350919,0.39560765,0.42359388,0.62181485,0.764327,0.18681261,0.18348885,0.67430085,0.2657509,0.66948104,0.8795041,0.57419854,0.5142833,0.4175837,0.3526397,0.6394266,0.31037688,0.7085169,0.15217544,0.656746,0.2506353,0.13130313,0.2604812,0.22549665,0.26683047,0.4037608,0.6783098,0.2640154,0.7025449,0.9925976,0.59470344,0.47485626,0.17064267,0.33647797,0.087206595,0.45357352,0.5257944,0.23288482,0.625595,0.87155336,0.30862406,0.73223364,0.5305823,0.8434149,0.9769529,0.1323935,0.44328752,0.6198008,0.19063587,0.1727799,0.077479094,0.6673898,0.016015766,0.6853796,0.42786616,0.9715973,0.8263588,0.23673232,0.86254555,0.11072131,0.97973096,0.47356296,0.8190762,0.570402,0.4867965,0.40925452,0.8916607,0.11200305,0.050857767,0.42794508,0.94161946,0.69716215,0.6087411,0.2817518,0.049653415,0.10126666,0.6541182,0.3597886,0.9665676,0.6147688,0.8397085,0.9083348,0.060538165,0.60607266,0.4794526,0.5722238,0.5808448,0.783964,0.01734779,0.6645924,0.93834233,0.751999,0.72635245,0.37987387,0.29888868,0.08431507,0.9688512,0.051511478,0.726588,0.7220806,0.088296264,0.28621355,0.61620796,0.44655597,0.9409492,0.669655,0.94399756,0.6191896,0.79607916,0.29986486,0.06823975,0.09386315,0.82962394,0.23778406,0.4734829,0.23600928,0.29363406,0.4395974,0.48661724,0.672904,0.68982255,0.42688656,0.7836344,0.20999289,0.8862822,0.80858916,0.60629743,0.84654224,0.027162872,0.20688719,0.67130744,0.5147718,0.8730931,0.48226056,0.061555773,0.62499255,0.8782004,0.41605252,0.441862,0.73870254,0.7205532,0.51237977,0.90391004,0.009413265,0.7498757]
fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	S10	[0.67345244,0.8998432,0.47292912,0.3351243,0.50291306,0.08474638,0.97303843,0.76687574,0.8329978,0.6745559,0.27237386,0.86640406,0.8440331,0.47707734,0.15656945,0.8384068,0.6988432,0.8042303,0.21771154,0.035301417,0.7292369,0.03284875,0.37321335,0.9896323,0.30047214,0.066865556,0.54811454,0.6670341,0.4506665,0.7330964,0.033810526,0.3278559,0.29196244,0.33999828,0.7730455,0.6432067,0.58849335,0.18444498,0.7668931,0.8065898,0.9728011,0.9243777,0.54887086,0.37211752,0.093007766,0.37022114,0.53427607,0.08557087,0.97668386,0.5081581,0.68720406,0.49504742,0.35932738,0.6824962,0.37208894,0.5595766,0.5801467,0.11561265,0.87388736,0.061645668,0.44722104,0.52741975,0.34540316,0.109274216,0.98203546,0.19953616,0.887276,0.641623,0.3339371,0.6123814,0.4072929,0.69850993,0.88587123,0.19947885,0.16598417,0.45575434,0.55935967,0.6679969,0.7745778,0.48813364,0.10542476,0.06951639,0.42853925,0.6930155,0.3139654,0.06833674,0.362305,0.6160775,0.9765963,0.29015797,0.8672031,0.96868676,0.5822815,0.71846086,0.4371354,0.9940525,0.9929879,0.9792942,0.9741519,0.85532856,0.031981546,0.4087518,0.55033875,0.5734395,0.26007098,0.30806166,0.7341058,0.8420618,0.87563765,0.7374785,0.32856378,0.9332427,0.8165286,0.12749301,0.71381164,0.92608887,0.7218311,0.48670584,0.9687231,0.010279691,0.75307727,0.9249441,0.09279975,0.5050936,0.63486606,0.25683665,0.42563537,0.33223704,0.005401269,0.78433496,0.052141372,0.79414487,0.1651352,0.032059073,0.6449386,0.55055845,0.0022013653,0.70481616,0.3389371,0.10306608,0.5020178,0.39676133,0.10201617,0.5713909,0.026101472,0.1679782,0.48530474,0.5859273,0.46138322,0.92932373,0.839103,0.24862549,0.39918172,0.44099554,0.7911856,0.103476495,0.41526264,0.13246347,0.92523324,0.4512268,0.48923665,0.6838237,0.02526382,0.13173391,0.800629,0.7286402,0.97474676,0.6032062,0.30222732,0.77916545,0.85546625,0.051900845,0.5400246,0.3434446,0.32279548,0.88273036,0.15970306,0.279958,0.79903364,0.7504278,0.42730433,0.09866553,0.8259699,0.3218635,0.5534846,0.6111585,0.06252565,0.81357926,0.6218801,0.21192329,0.21269456,0.9682061,0.85014886,0.94207734,0.6619276,0.387736,0.049676806,0.90967995,0.7403628,0.8570942,0.981298,0.82904065,0.20147213,0.10531816,0.94127196,0.38832682,0.2009691,0.52263415,0.9506974,0.013152694,0.28953767,0.86036766,0.5936572,0.14195226,0.92236257,0.7523128,0.10966478,0.108672984,0.67796564,0.9044355,0.31997204,0.8695624,0.108136505,0.8601999,0.59407604,0.92667824,0.48068836,0.94357365,0.9360459,0.30113783,0.1505863,0.022002498,0.041491363,0.95934063,0.10616043,0.413575,0.16852118,0.8896107,0.20326832,0.017761718,0.68373865,0.2596067,0.649575,0.5070684,0.5445686,0.71889824,0.3533798,0.97261566,0.070177354,0.91582847,0.6812251,0.028882517,0.91943896,0.850556,0.66037184,0.004351305,0.7928936,0.002825572,0.41162682,0.05608811,0.1625419,0.7816758,0.75870466,0.3606072,0.009120109,0.5329253,0.080315836,0.9217697,0.22624867,0.6007501,0.2822429,0.44002512,0.41981027,0.5126193,0.4949878,0.8120051,0.12706207,0.8707728,0.4663885,0.92601633,0.31104204,0.9622019,0.7840428,0.09431316,0.37580568,0.6696635,0.6655076,0.9944482,0.47314408,0.71590817,0.9925887,0.21263184,0.33603296,0.68961877,0.35778975,0.9037355,0.27896547,0.5843781,0.10677798,0.2708293,0.36673352,0.10218525,0.11463001,0.6947953,0.3335228,0.331365,0.8476106,0.2079979,0.63998467,0.5837356,0.12089193,0.9573513,0.7784006,0.42278185,0.0052604247,0.44530797,0.48596025,0.3922571,0.5097875,0.33914638,0.6476089,0.04872271,0.38883308,0.6382922,0.033644684,0.23001774,0.22871724,0.39403445,0.06479122,0.16318673,0.6881583,0.4295925,0.36452615,0.5303612,0.7827923,0.56432945,0.25535414,0.57331437,0.1036911,0.21428087,0.8676931,0.78395665,0.7301622,0.33004424,0.6604933,0.6583397,0.74058706,0.768079,0.62933505,0.22861403,0.9136441,0.2426559,0.46580496,0.29104698,0.9724797,0.88198704,0.55229264,0.17905547,0.23319021,0.6618984,0.7139528,0.5976867,0.03057148,0.7447508,0.68271554,0.6749303,0.16006427,0.13319874,0.10366592,0.63649565,0.3025764,0.19671634,0.6425595,0.6323407,0.19894141,0.9683416,0.2540223,0.10750834,0.11622819,0.5515973,0.9582513,0.9179972,0.6613237,0.5663692,0.068827905,0.522624,0.52702504,0.13737026,0.9061629,0.5484852,0.72958726,0.40659,0.33788761,0.7953658,0.4315913,0.14617616,0.1502023,0.16657564,0.9991856,0.25149202,0.21543531,0.6721397,0.38448215,0.72370476,0.54298645,0.8828227,0.5716989,0.6282845,0.89805067,0.5847877,0.10354698,0.04398487,0.05523436,0.5738944,0.3645588,0.6633839,0.047102667,0.58803815,0.23361468,0.16548428,0.3561888,0.459162,0.65986687,0.74852806,0.9551753,0.036237113,0.23329872,0.36409214,0.70440716,0.9547846,0.13770045,0.4300998,0.4789015,0.19175406,0.22050269,0.2958842,0.12037865,0.2308755,0.34191266,0.3617992,0.7903768,0.32277188,0.057320442,0.9214829,0.97420394,0.7357195,0.73058957,0.027207637,0.7141448,0.4541964,0.338375,0.306019,0.41640145,0.35619128,0.9903935,0.23437133,0.783099,0.7293635,0.1934395,0.012666678,0.7098263,0.5301659,0.47748908,0.60069734,0.74977577,0.26092952,0.27008885,0.2373526,0.097492956,0.9895202,0.56433994,0.13732116,0.6620798,0.8311281,0.5737184,0.011082939,0.23460779,0.6066982,0.42133397,0.023300262,0.9750081,0.9184805,0.18530679,0.32614067,0.30471447,0.05528885,0.6846527,0.84684324,0.10082315,0.7768075,0.15370509,0.82626384,0.71151614,0.7116064,0.2323496,0.7177121,0.6839513,0.12921394,0.57129836,0.25280714,0.6206748,0.85018176,0.22328414,0.105789006,0.00054593553,0.42571887,0.33934623,0.35454333,0.82493836,0.76696026,0.8303189,0.06419772]
70c790b5-8ebd-4593-bf15-20bc404d408f	S11	[0.70071137,0.3835612,0.24079214,0.2844776,0.008127679,0.56255955,0.9773548,0.5379967,0.20517534,0.024878385,0.76802546,0.45008036,0.08023787,0.9980104,0.9451729,0.7611108,0.87356484,0.29102772,0.17687108,0.01305534,0.46412182,0.28580478,0.374968,0.39257637,0.23336753,0.6276331,0.66983205,0.72720796,0.78045774,0.5681495,0.33052173,0.46924523,0.70305276,0.13052273,0.9389351,0.15082169,0.7480477,0.6324967,0.81058854,0.4315135,0.21507786,0.6830552,0.2408142,0.478201,0.3596552,0.44911852,0.61176324,0.72769195,0.8190775,0.38676944,0.22360641,0.7739203,0.8643776,0.8081121,0.44945705,0.55215114,0.51454157,0.084855475,0.6430136,0.7010506,0.78985614,0.278327,0.45690906,0.23475696,0.25245592,0.5796282,0.14462958,0.96825624,0.9552865,0.67178535,0.99091226,0.19963893,0.03546009,0.11799671,0.40832785,0.15815651,0.7665555,0.9600644,0.98973155,0.5207365,0.65412694,0.12051973,0.05132881,0.16707171,0.13719201,0.32519457,0.10633277,0.7351755,0.16259798,0.93995124,0.28363878,0.7505057,0.63975006,0.39682746,0.44502273,0.6869743,0.954778,0.15934375,0.8722267,0.19448052,0.24300542,0.9485708,0.77683085,0.5815804,0.9152828,0.9587866,0.16661444,0.20811306,0.5135493,0.3766643,0.4283712,0.04406168,0.194457,0.06326033,0.32770437,0.73648286,0.9624619,0.2728207,0.62986386,0.61428,0.18608887,0.6413388,0.67072314,0.60610324,0.8058719,0.632756,0.24691157,0.6587649,0.6295514,0.6283979,0.20929325,0.98209304,0.5005082,0.4178651,0.41284785,0.2615932,0.5957229,0.06916655,0.5906777,0.02900589,0.34837803,0.09006192,0.35870293,0.53756255,0.8818549,0.652242,0.64989126,0.0923264,0.34537002,0.9291566,0.4944156,0.84001565,0.12355057,0.4251012,0.8009247,0.36820325,0.39821306,0.58389527,0.5073668,0.4407327,0.43629947,0.9756392,0.8886216,0.24953201,0.53078985,0.57464397,0.6612509,0.5735919,0.2632523,0.17820959,0.7247922,0.927031,0.65737677,0.52125126,0.6602392,0.8659397,0.48858675,0.9270797,0.78513366,0.8870335,0.05697949,0.7993064,0.77609193,0.17558973,0.20607933,0.17132385,0.44929472,0.9621786,0.8466063,0.11187835,0.20915923,0.5916096,0.672228,0.41127682,0.22547764,0.64095443,0.78557676,0.48825452,0.5207751,0.73311365,0.6670485,0.68901724,0.4397851,0.68668145,0.08967707,0.21724474,0.8182946,0.65970373,0.8798995,0.2266676,0.38667625,0.6895048,0.2261175,0.6590222,0.47528094,0.16427056,0.40687048,0.2852272,0.827497,0.54395527,0.43027452,0.5288021,0.020702887,0.6138946,0.14006157,0.70300066,0.44623405,0.6019297,0.7053751,0.07686626,0.31389436,0.6263189,0.26220825,0.07982998,0.11081764,0.9591943,0.04527488,0.82082206,0.1535739,0.062105134,0.5563465,0.5089035,0.84477746,0.59135556,0.5221894,0.9011582,0.89831424,0.7928999,0.27484095,0.9086268,0.675606,0.8963617,0.13637857,0.13329108,0.612978,0.28226963,0.054535266,0.51595664,0.7719106,0.2527381,0.004800982,0.008814365,0.28763217,0.6358728,0.527002,0.2113734,0.13495615,0.7246446,0.4407673,0.6886476,0.28646865,0.37859657,0.04896839,0.74217254,0.93573225,0.98369193,0.8480279,0.13344699,0.41618285,0.70813113,0.21936071,0.9004438,0.9600205,0.21027754,0.7644879,0.437615,0.4159062,0.97917134,0.20791638,0.71469486,0.5902525,0.26307145,0.20494023,0.86018765,0.17246729,0.9266317,0.5522578,0.78821975,0.90401965,0.6366864,0.4179036,0.26378196,0.23321037,0.5643314,0.49504265,0.8858694,0.09816862,0.9838625,0.7420102,0.552429,0.81686556,0.45365927,0.2211496,0.46292403,0.8256861,0.038089335,0.043501016,0.37957305,0.049649335,0.6139982,0.3750258,0.65065986,0.4038409,0.5626191,0.13541889,0.25867864,0.10154581,0.20461002,0.336,0.35988006,0.11716999,0.99041605,0.9473645,0.20011573,0.68547577,0.44471607,0.12346602,0.38172683,0.31742123,0.04910563,0.56593716,0.960551,0.27124435,0.6484421,0.30382448,0.50835824,0.6110973,0.83123624,0.35107833,0.37285566,0.3016388,0.6943746,0.19767608,0.2706713,0.004170476,0.11386516,0.67835623,0.8611575,0.718613,0.35724837,0.17504542,0.057958167,0.95080614,0.77375525,0.16856778,0.52941895,0.866009,0.025713604,0.7952605,0.024925424,0.46958163,0.24485329,0.7356874,0.9089187,0.112075254,0.6574089,0.5690478,0.8426983,0.517811,0.3507805,0.4990146,0.9067703,0.2811319,0.615477,0.7375084,0.53960824,0.043828726,0.9812712,0.32148647,0.2504376,0.51074994,0.24804503,0.84794164,0.6836951,0.76794434,0.555367,0.26478636,0.42098638,0.6912111,0.09210535,0.9201763,0.47317308,0.97930473,0.44469905,0.66546434,0.8608069,0.39778534,0.46537235,0.41094717,0.06760748,0.64903337,0.06803743,0.04976841,0.00015692782,0.1755704,0.33211228,0.6409022,0.65428394,0.56362,0.24886991,0.41211402,0.22741489,0.5233648,0.011872914,0.6207431,0.3544341,0.14036997,0.7000316,0.4206706,0.5921872,0.48218933,0.2390152,0.6744731,0.27338588,0.8061791,0.3637057,0.19884275,0.11090629,0.13258287,0.56616324,0.30373612,0.81646687,0.9034475,0.51429635,0.08292838,0.74244374,0.9746521,0.22788188,0.79205537,0.53220755,0.69293493,0.19316761,0.4409123,0.08848989,0.017810047,0.691187,0.7945083,0.17489026,0.07756685,0.63429964,0.15500711,0.673112,0.7419117,0.67857873,0.91622186,0.8696317,0.23935302,0.5278179,0.54957515,0.79579765,0.060941596,0.021511877,0.36095214,0.53295773,0.3694091,0.87466556,0.12643097,0.25192305,0.55083275,0.34121326,0.82591105,0.27291813,0.6972499,0.0051569007,0.86861664,0.40139297,0.073823124,0.4155222,0.1827737,0.023663916,0.24731107,0.9582852,0.9528769,0.28943202,0.2551171,0.8659812,0.3064237,0.8626208,0.81060255,0.98569804,0.542144,0.46396905,0.099531054,0.6780548,0.7500999,0.55325013,0.8091863,0.6010684,0.67845714,0.50357044,0.05663118,0.06375914]
04786799-ca23-4231-9cb1-527b8dd7e154	2111818	[0.01359,0.00043222,0.048354,0.021453,-0.004545,-0.095706,0.0045633,0.012375,-0.03739,-0.047124,0.029227,0.03615,0.056727,0.002464,-0.00087406,0.043091,-0.017594,-0.046793,0.046017,-0.044943,0.0053107,-0.089208,-0.042859,-0.1162,-0.027617,0.051027,-0.0025608,-0.010628,-0.016793,0.0687,-0.057149,-0.058192,-0.072978,0.053014,0.087719,-0.045666,0.069955,0.10312,-0.0068954,0.014371,0.017875,0.0099112,-0.047836,0.01314,-0.059167,0.0054838,0.078221,-0.029606,-0.017873,-0.040315,-0.02907,0.046252,-0.046601,0.0077382,0.028164,-0.12915,-0.083684,0.10723,0.02313,-0.041921,0.13457,-0.0916,0.023915,-0.033909,0.032588,0.081918,0.035901,-0.010277,0.037745,0.032925,0.032253,-0.014519,0.060473,0.11211,-0.019585,-0.041234,0.022934,-0.009187,-0.067636,0.02963,0.016559,0.0094523,0.046937,0.039923,-0.021773,0.012144,-0.028035,-0.074011,-0.0072655,0.044705,-0.0038219,-0.011298,-0.0026093,-0.032941,0.033237,-0.0060467,-0.065126,-0.02399,-0.012478,0.040298,0.004523,0.022413,0.047957,-0.00641,-0.0015143,0.017964,-0.011745,-0.023654,-0.042666,0.046592,0.10532,0.027171,0.063569,0.00764,0.07395,0.047235,0.0088586,0.033334,0.00059348,-0.0095075,-0.033297,-0.0023221,0.024593,-0.034492,0.00041257,0.053036,-0.015622,0.035427,0.021448,0.02628,-0.027233,0.095594,-0.039277,0.026522,-0.044264,0.046224,-0.04369,0.047873,-0.0067438,0.0059344,0.046574,-0.020197,0.039873,-0.0277,0.056166,0.018398,-0.057384,7.6307e-05,-0.036784,-0.036367,0.032043,0.066072,0.017744,0.050961,-0.0004515,0.0052124,0.019374,0.008347,0.016186,0.05762,0.023972,0.042684,0.014802,-0.025972,0.021458,-0.040918,0.06896,0.063223,0.011771,0.011876,0.071673,-0.00439,0.0041854,-0.037733,-0.017993,-0.031768,-0.06223,-0.006027,0.09781,0.11165,0.0523,0.06793,-0.031908,0.041595,-0.032024,0.017161,-0.016368,0.033827,-0.048863,0.0063264,0.038868,-0.067716,-0.035601,0.0032481,0.0055594,-0.015806,0.00034336,-0.06604,0.062885,0.050351,-0.037758,-0.036659,-0.039713,0.0073913,-0.028082,-0.014306,-0.045868,0.010022,0.0084942,0.043433,0.015183,-0.0016091,-0.018674,0.017193,0.012191,0.036654,-0.021735,0.033242,-0.045331,0.0021936,0.019185,-0.056792,-0.011684,-0.029584,-0.010939,-0.018574,0.04955,0.047545,0.023938,0.035168,0.014387,-0.040201,0.0039997,-0.060162,-0.028502,-0.046332,0.010485,-0.032214,-0.016979,0.045312,0.016039,0.026639,-0.0043238,0.058095,-0.0043109,-0.044564,-0.049996,-0.038589,-0.038603,0.025653,-0.079984,0.088934,-0.014887,0.0024891,-0.03824,-0.0044265,-0.043928,0.0030916,0.003145,0.02461,0.006637,0.090109,0.019239,0.0057171,0.017145,-0.036072,0.014368,0.066467,-0.025717,-0.040895,-0.042417,0.0042031,-0.041278,-0.041107,-0.0206,0.036988,-0.02517,-0.018624,-0.03558,0.011767,-0.039133,-0.026163,-0.054895,-0.017532,-0.039432,-0.10218,0.070177,-0.000595,0.042074,-0.00028558,0.02583,0.0018844,-0.010177,-0.080357,0.0059134,-0.051881,-0.0062365,-0.039514,0.017361,0.01402,0.0275,-0.017918,0.031502,-0.0043283,0.0076051,0.0021306,0.10852,0.0040652,-0.033442,0.052061,0.084245,-0.053835,-0.015381,-0.052801,-0.021344,-0.02434,0.014905,-0.0051427,-0.054723,-0.027641,0.024738,0.053018,0.034233,0.017619,0.063352,-0.0073014,-0.043211,-0.12311,0.093132,-0.0072117,-0.03539,0.072196,-0.041303,-0.055113,-0.037199,0.052615,-0.010327,-0.015086,0.10546,-0.011565,-0.020778,-0.095819,-0.071397,0.007947,-0.026638,0.023772,0.017035,0.094708,0.094997,-0.068339,-0.037123,-0.0018051,-0.021042,-0.025498,-0.011324,-0.0041767,-0.031495,-0.021343,0.042883,0.037704,-0.002038,-0.04026,0.028992,0.073034,-0.018416,-0.0012088,0.030843,0.050323,0.015276,0.085866,0.054945,-0.038258,-0.024016,-0.021007,0.067428,-0.062337,0.02929,0.06234,-0.0075058,0.011587,0.020009,0.0058545,-0.016533,0.021681,-0.045426,-0.030651,0.0087955,-0.05297,-0.054561,0.0095399,-0.016507,-0.014889,-0.086462,0.017523,-0.0083427,0.0023548,0.017734,-0.00059559,-0.022105,-0.043656,-0.01449,0.15713,0.018843,-0.023115,0.01321,-0.02832,-0.0030586,-0.076291,-0.049915,-0.027377,-0.018384,0.022196,0.037684,0.019169,-0.047422,-0.029999,-0.015662,0.066568,-0.0071747,-0.00036978,-0.020268,-0.0037887,-0.046294,0.020401,-0.0035,0.010726,-0.056744,0.042163,0.035199,0.041646,-0.01556,0.0088996,0.03456,-0.042617,-0.026601,-0.015352,0.073452,-0.0064133,0.16683,0.07051,-0.093725,-0.036823,0.0071502,-0.0075379,0.022751,0.041701,-0.049505,0.087479,-0.015552,0.0078552,0.047152,-0.087859,-0.093182,0.041079,0.015935,0.0080382,0.030954,0.014053,0.0055103,0.063941,0.0011108,-0.017267,-0.028461,0.0093821,0.028654,0.018422,0.027298,-0.043749,-0.028409,0.041252,0.064767,-0.047117,0.0023974,-0.018916,-0.0031734,-0.022838,-0.04039,0.050562,0.036561,-0.029723,0.018824,-0.028547,0.035477,0.011381,-0.040984,0.094059,0.0071482,-0.10697,0.01936,-0.033039,-0.0041905,0.022147,0.027785,0.016645,-0.060856,0.072183,-0.022535,0.017256,0.010043,0.056018,-0.023503,-0.078512,0.041692,0.054609,0.037115,0.068727,-0.036176,0.016365,0.098682,0.017311,-0.056415,0.045993]
62e33c55-0d38-4a0b-87cc-468038adf30e	S12	[0.1886965,0.8313747,0.53392977,0.5218009,0.78218174,0.7539967,0.95373195,0.5423867,0.14573912,0.43740812,0.7601694,0.74532986,0.55398196,0.18648991,0.6134941,0.60231674,0.6273667,0.61123776,0.75864124,0.36507896,0.13993448,0.38248846,0.76908344,0.85483146,0.28851527,0.009290164,0.9467046,0.97647643,0.02107307,0.8508621,0.76597345,0.32145876,0.591438,0.7296619,0.5152614,0.70515436,0.13717611,0.58769375,0.06687282,0.98251164,0.79826176,0.66423833,0.58766323,0.82169837,0.2218216,0.6441189,0.9967103,0.5097711,0.112913705,0.063932344,0.4302904,0.19710504,0.57777286,0.83778924,0.8538327,0.070209354,0.2708376,0.3098054,0.42088893,0.41913202,0.49400812,0.15011525,0.14647003,0.68662184,0.6900287,0.9306181,0.49520606,0.00805079,0.6144942,0.07548278,0.6316922,0.7582208,0.9739645,0.96372503,0.6058409,0.28026327,0.56008,0.41067725,0.21534908,0.5913405,0.5968408,0.8082013,0.99087024,0.9366243,0.42742264,0.1993765,0.77251977,0.65602005,0.06332973,0.8790671,0.6292346,0.24598421,0.73627466,0.69182104,0.2048978,0.005032596,0.16216367,0.8470343,0.464526,0.8483348,0.14600089,0.7526496,0.47476828,0.3392221,0.9941091,0.11303292,0.08054551,0.2912651,0.6768676,0.79013336,0.9477384,0.91637945,0.61136675,0.5697819,0.7694371,0.8175891,0.45394984,0.42047027,0.7010725,0.10755611,0.7674048,0.19929565,0.003087632,0.8950086,0.38514298,0.805074,0.17101368,0.18061079,0.8930778,0.28209049,0.5300777,0.9062018,0.6126519,0.970494,0.6495745,0.9137154,0.94943804,0.76110643,0.03463093,0.620468,0.528337,0.53728545,0.52668756,0.75496423,0.881922,0.6295045,0.25293726,0.013992614,0.0026381023,0.6215486,0.58695126,0.4826474,0.72039723,0.112794496,0.9430061,0.09281981,0.87011504,0.4101168,0.0032986547,0.26716605,0.6845483,0.387531,0.9867874,0.9007755,0.39346167,0.059196483,0.57761204,0.37260824,0.020556359,0.097196,0.31412748,0.21038578,0.8232533,0.36348888,0.17605378,0.46954072,0.3799382,0.7115425,0.59492886,0.69816107,0.64847815,0.28943247,0.44799668,0.6324575,0.9749492,0.3879464,0.861816,0.36936137,0.8261031,0.6046074,0.16398047,0.0065884995,0.20024638,0.5597519,0.9718952,0.97821635,0.021280529,0.87593174,0.19023807,0.67348915,0.30191824,0.31124583,0.13190621,0.17199433,0.15904738,0.8519124,0.5503681,0.48375368,0.7095313,0.16804576,0.46397662,0.5203218,0.003146414,0.21026775,0.8287179,0.25269958,0.9119098,0.6586948,0.642498,0.6708344,0.36549863,0.036712702,0.74242496,0.87530583,0.2626488,0.19442768,0.9338904,0.370775,0.21864313,0.65799314,0.3947783,0.7594742,0.5768438,0.90445393,0.706753,0.3956204,0.22584423,0.7369849,0.3687568,0.5238873,0.29657874,0.077661574,0.87240666,0.08442555,0.8770237,0.46812543,0.42547464,0.4491974,0.42444918,0.44486985,0.21980077,0.13374238,0.45743695,0.3226848,0.03763896,0.12804228,0.44150075,0.13653894,0.48775724,0.08773614,0.64192384,0.7603997,0.3475385,0.24128945,0.19834724,0.8789252,0.99902123,0.20708945,0.30622986,0.6623419,0.5024435,0.2424954,0.23338805,0.3082838,0.7745853,0.961194,0.7340071,0.4318397,0.18086377,0.71308726,0.53154886,0.50093204,0.6784192,0.96250075,0.6212191,0.651164,0.5589917,0.17360757,0.31588507,0.8650093,0.83451515,0.9371407,0.8141572,0.634832,0.2528251,0.92697006,0.5909589,0.81632024,0.4592178,0.4724753,0.9406206,0.57672083,0.85298324,0.022274675,0.57811016,0.20301837,0.00017662387,0.08584177,0.101800375,0.16603312,0.73282695,0.58593607,0.2944586,0.19475307,0.056705836,0.38959664,0.42326912,0.118986495,0.9536385,0.13114406,0.21761695,0.28958172,0.62543654,0.7611709,0.4559218,0.50717556,0.7737523,0.38656896,0.09955984,0.33863956,0.63846135,0.9173747,0.5321179,0.32815528,0.31497332,0.28238806,0.13310836,0.037814762,0.8927545,0.21899493,0.62635005,0.82171935,0.78133297,0.44945577,0.77498025,0.35091105,0.28961813,0.03421155,0.47080147,0.59489423,0.08151419,0.40935758,0.8001994,0.54998934,0.6496056,0.32127902,0.48240182,0.41158563,0.94078475,0.06949806,0.51414245,0.00945936,0.8215039,0.34081927,0.29761076,0.3358883,0.49225459,0.035177987,0.6057841,0.93236923,0.827099,0.12462612,0.93628013,0.1486153,0.27259004,0.8309146,0.33778998,0.95695037,0.31309652,0.21396196,0.8997391,0.8402005,0.7839718,0.9968198,0.666473,0.66840416,0.68639237,0.18377696,0.29164588,0.5798195,0.7639293,0.91501296,0.2926641,0.5314153,0.8476391,0.96675545,0.27709433,0.4666747,0.31509253,0.0048993803,0.32741338,0.11623344,0.98338026,0.32847068,0.8103473,0.45494604,0.29747945,0.997306,0.88193136,0.37949654,0.10949601,0.74118495,0.1390896,0.5564964,0.054013107,0.1633646,0.48633638,0.37059563,0.093816884,0.6529207,0.14834401,0.3835264,0.9574356,0.5667451,0.51633173,0.19896798,0.5733341,0.13207944,0.6729099,0.56177634,0.18718126,0.625278,0.4819476,0.38844925,0.30051807,0.025758108,0.6032774,0.84959984,0.49337977,0.9493578,0.0694114,0.6385262,0.4438968,0.10056861,0.0905115,0.2768413,0.423419,0.8863821,0.08571501,0.86278373,0.46944505,0.6908535,0.12717792,0.1332195,0.21673876,0.15685971,0.9120894,0.3748408,0.17328094,0.69840777,0.28030843,0.67454594,0.0614679,0.7149877,0.9556479,0.78677046,0.8602486,0.4281034,0.954127,0.85654813,0.3378771,0.6964541,0.83019245,0.23728634,0.9910953,0.5658724,0.30907273,0.40379298,0.6778868,0.4933536,0.76736784,0.047121506,0.56980896,0.4790047,0.7166486,0.7331995,0.5848693,0.70322835,0.68638915,0.98376536,0.3837259,0.8693905,0.72637254,0.911777,0.18378943,0.9627453,0.54131293,0.9480526,0.1387277,0.040317528,0.2658942,0.110849686,0.90998906,0.27433303,0.52845347,0.052844513,0.38040817,0.2320754,0.7418122,0.92900425,0.9426781,0.14083621]
d64d1501-42f4-4685-b7fe-1f9785f4ece8	S4	[0.39106324,0.08023228,0.8801688,0.5848874,0.6462766,0.5893364,0.39632,0.5313216,0.5780564,0.31187022,0.66202605,0.96784955,0.418515,0.1343659,0.38159078,0.5287342,0.43067497,0.10292367,0.94948983,0.23510964,0.20629399,0.41070366,0.31320667,0.9604478,0.5129191,0.16560434,0.68582356,0.98699856,0.8558768,0.505959,0.8273085,0.2865952,0.20086771,0.5180206,0.12531589,0.58789057,0.9428221,0.60977495,0.4346037,0.5163497,0.20255387,0.59851307,0.36172813,0.04499359,0.96925014,0.38892907,0.07256238,0.15147492,0.8957409,0.9938616,0.55565256,0.6611464,0.78899163,0.97514373,0.8531733,0.90501505,0.081210755,0.9495289,0.49653557,0.10464277,0.36432537,0.31888032,0.32762596,0.075339645,0.45706138,0.38412195,0.46498898,0.47379568,0.28182942,0.19107929,0.8439361,0.16040616,0.7819441,0.9287941,0.93723357,0.42994875,0.064161755,0.31875885,0.08882698,0.84220356,0.20480627,0.67708933,0.76931924,0.56843096,0.5995327,0.20322996,0.79530674,0.77931476,0.8548415,0.46152255,0.1068994,0.9467194,0.14690411,0.4585032,0.97346985,0.44582528,0.10360499,0.41824403,0.7900148,0.18755971,0.5702397,0.58028406,0.24857008,0.6239271,0.2900328,0.5441728,0.19585864,0.4819582,0.34935218,0.6297826,0.32189023,0.041475505,0.91713065,0.90922326,0.99025315,0.6786349,0.8364575,0.66102844,0.8323947,0.01983235,0.46375418,0.31860638,0.13486521,0.53068954,0.50480014,0.63613313,0.38297185,0.3893608,0.8971492,0.641857,0.4547772,0.4045668,0.46700886,0.45635653,0.82471627,0.1177989,0.13698882,0.5982366,0.34189832,0.0641889,0.105615355,0.25282863,0.16414784,0.63698936,0.8148732,0.040887285,0.88423485,0.6737819,0.3956881,0.6012555,0.98268056,0.8529081,0.0475801,0.8085848,0.019300243,0.49035916,0.8273191,0.9937169,0.6354877,0.80467254,0.8832905,0.41101304,0.25394034,0.17913403,0.6064351,0.42291236,0.22805785,0.7519864,0.7495457,0.4342614,0.22012492,0.14896952,0.9620048,0.80208766,0.9716708,0.970411,0.8706789,0.12640953,0.17357808,0.37168518,0.4594014,0.6685386,0.78484774,0.9991838,0.33938074,0.73377055,0.5465015,0.29009116,0.26753795,0.82631904,0.60927856,0.01893623,0.96559864,0.7029481,0.29457656,0.56734073,0.16598833,0.7532055,0.61013263,0.070375025,0.48039517,0.95833844,0.5048824,0.035006627,0.5323873,0.8391225,0.5052148,0.55434686,0.81934226,0.9346442,0.76335645,0.7013398,0.27396312,0.37278596,0.48416868,0.7131311,0.8776429,0.03179275,0.21155986,0.7998482,0.9803219,0.30547684,0.12630515,0.4685187,0.4841477,0.96192926,0.9964236,0.07402172,0.8260341,0.74905056,0.080998145,0.31304535,0.5277721,0.0024627175,0.6826632,0.63554317,0.24267407,0.39328206,0.94227785,0.69762945,0.8788879,0.53874165,0.9869531,0.27146384,0.45478818,0.15240227,0.5452429,0.1618643,0.6366085,0.1123212,0.18722622,0.4080955,0.6565467,0.46504387,0.8788256,0.38979718,0.38195083,0.67282856,0.9300015,0.9830106,0.051949915,0.45139864,0.99110883,0.37931946,0.72238433,0.6133108,0.797052,0.6101238,0.2553281,0.49722454,0.58751005,0.4931718,0.46533066,0.18663915,0.5373222,0.10976511,0.65556425,0.38589495,0.85587764,0.50641996,0.23276863,0.220768,0.16380003,0.32018986,0.87354326,0.9751693,0.85396415,0.4074857,0.73321074,0.7857181,0.4130501,0.99453175,0.42898592,0.4758464,0.88148344,0.62397623,0.90940106,0.17096134,0.78977394,0.7675282,0.33848768,0.7923482,0.37593213,0.11358334,0.16981922,0.9713869,0.37888715,0.12641317,0.2994697,0.3428551,0.83839476,0.44216844,0.25844768,0.5753213,0.7023515,0.1332786,0.9533612,0.638809,0.730705,0.40423533,0.481794,0.65750456,0.8257865,0.0071659614,0.4993299,0.6467448,0.35883367,0.09377413,0.2482143,0.47102928,0.53633374,0.2656757,0.30589992,0.5000315,0.62059075,0.41933778,0.31770602,0.27829218,0.44413173,0.7188513,0.09812464,0.26359344,0.44265953,0.28887376,0.93515325,0.32372254,0.34994882,0.74913555,0.2868694,0.84161806,0.0023900305,0.20886719,0.62601036,0.25957993,0.61070687,0.9181341,0.71521515,0.720728,0.31579152,0.9183042,0.12574658,0.10030666,0.3637336,0.10108708,0.6185957,0.31728047,0.11640245,0.098679975,0.008863644,0.57115513,0.5558121,0.3078991,0.9810784,0.76537865,0.4043986,0.78794926,0.9588097,0.57328564,0.018684814,0.5672535,0.36084253,0.37893218,0.3651316,0.5134677,0.36828482,0.4677215,0.84102017,0.12602064,0.7622391,0.33234847,0.6067493,0.6355422,0.37534934,0.010221563,0.95208955,0.13955437,0.99981964,0.2166218,0.6058179,0.02941401,0.09609102,0.89069074,0.9802785,0.52752227,0.20566168,0.81168574,0.37052956,0.901474,0.66887295,0.119107634,0.7134585,0.8323607,0.6498202,0.508615,0.8598351,0.8764547,0.3703283,0.2713967,0.3345948,0.1515468,0.9584818,0.704291,0.96821827,0.062348455,0.9338671,0.42147434,0.6524669,0.61056113,0.043473132,0.13908128,0.6890987,0.2454391,0.5852453,0.46139959,0.04466556,0.9151837,0.71739423,0.93081677,0.59278977,0.50760925,0.06889898,0.8572884,0.93816704,0.8943476,0.6864075,0.64058,0.21763252,0.72405505,0.49330547,0.46830243,0.038667902,0.24111705,0.41477713,0.19241816,0.34252268,0.4877149,0.97732407,0.424555,0.24709167,0.45411214,0.37805787,0.26922727,0.04443074,0.9680869,0.8239417,0.102253735,0.8700934,0.43834952,0.52153236,0.31006217,0.4390009,0.38400525,0.76629955,0.9322823,0.63257164,0.76919085,0.026490899,0.68469733,0.38609785,0.77842176,0.21535881,0.3985014,0.09292824,0.7979793,0.54838276,0.05569268,0.94834954,0.71172374,0.05672467,0.70779526,0.6648315,0.2905164,0.40237388,0.4835274,0.20883472,0.9188631,0.015928248,0.43445984,0.9120281,0.30494022,0.26795968,0.69263285,0.32960826,0.603274,0.51983416,0.72063375,0.3005821,0.44494516,0.27850646,0.4838949,0.8898775,0.8513943]
\.


--
-- TOC entry 3808 (class 0 OID 16716)
-- Dependencies: 216
-- Data for Name: users; Type: TABLE DATA; Schema: public; Owner: postgres
--

COPY public.users (user_id, email, first_name, last_name, role, password_hash, created_at, updated_at, username, image_url) FROM stdin;
525fe628-c06f-413e-af2f-b421e5bdcc16	student2@example.com	First2	Last2	student	hashedpassword2	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
bffc211b-88e3-4a54-9286-01511c608b0e	student13@example.com	First13	Last13	student	hashedpassword13	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
25b90d1d-4e1d-48c8-adcb-a334c517fc2d	student14@example.com	First14	Last14	student	hashedpassword14	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
821dee23-eaa0-447d-9aa3-f25b6570f98d	student15@example.com	First15	Last15	student	hashedpassword15	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
1dde1982-6705-4d6f-8f47-ddc1cfa83168	student16@example.com	First16	Last16	student	hashedpassword16	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
bfea3fec-742d-4cab-abbc-a0fd90827cb1	student17@example.com	First17	Last17	student	hashedpassword17	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
b6c748f3-d21c-48e4-a975-f604ebd9240a	student19@example.com	First19	Last19	student	hashedpassword19	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
29a519d0-5c29-4496-9687-1b7942dbd7ff	student20@example.com	First20	Last20	student	hashedpassword20	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
9977e3f2-a43f-4e5a-9115-6ba44bc54cec	student21@example.com	First21	Last21	student	hashedpassword21	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
b2d261b4-3b33-4d88-9cd9-7cd634872290	student22@example.com	First22	Last22	student	hashedpassword22	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
ff06e4d2-943b-4807-a3d7-395df26aa888	student23@example.com	First23	Last23	student	hashedpassword23	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
22309dfb-85ce-4221-beda-c12796788357	student24@example.com	First24	Last24	student	hashedpassword24	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
1eac1149-9c04-4934-93f9-74f2e7fc7fd3	student25@example.com	First25	Last25	student	hashedpassword25	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
50acc0da-9191-4e0d-b2f1-f95f49e85c8a	student26@example.com	First26	Last26	student	hashedpassword26	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
d9445cb8-2348-433e-ac8b-3af85feb6469	student27@example.com	First27	Last27	student	hashedpassword27	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
a416164c-78e4-46dd-8038-3c81b336190f	student28@example.com	First28	Last28	student	hashedpassword28	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
44bd3826-9511-411c-aa62-0b12e38584ed	student30@example.com	First30	Last30	student	hashedpassword30	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
98b24b88-7c84-4d9e-b8a9-35060056f170	lecturer2@example.com	First2	Last2	lecturer	hashedpassword2	2025-03-13 07:01:35.583166+07	2025-03-13 07:01:35.583166+07	\N	\N
c5dca646-dc37-413e-8db4-61f9a06b1284	lecturer3@example.com	First3	Last3	lecturer	hashedpassword3	2025-03-13 07:01:35.583166+07	2025-03-13 07:01:35.583166+07	\N	\N
5f7d2b42-0198-4871-b364-f9c83a34bd27	lecturer4@example.com	First4	Last4	lecturer	hashedpassword4	2025-03-13 07:01:35.583166+07	2025-03-13 07:01:35.583166+07	\N	\N
850a0e1c-3107-454f-9d46-211a1a92491b	lecturer5@example.com	First5	Last5	lecturer	hashedpassword5	2025-03-13 07:01:35.583166+07	2025-03-13 07:01:35.583166+07	\N	\N
04786799-ca23-4231-9cb1-527b8dd7e154	student18@example.com	Vũ Bá	Đông	student	hashedpassword18	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
ca6e3e92-e678-4af1-b69b-e6e73bf5f5d5	student01@example.com		Doe	student	$2a$10$2ekHEWBYKfKOmSYWiTkNNOe9INRQm5SgVjHvWYZKDN0w3yh5zLGlS	2025-03-21 10:46:17.421984+07	2025-03-21 14:10:28.018772+07	student01	\N
87cd50fe-5b68-4e30-9165-f4ae65303907	2111002.com	Linh	Nguyen	student	$2a$10$Av8KPZeeCVabZ/eD.1ebde1vLDi9.fVHDudOPA/JYgiC/5nhgttVW	2025-03-21 10:56:04.867196+07	2025-03-21 16:37:00.848602+07	this_vannam	https://example.com/avatar.jpg
19db57b6-55bb-431d-97e8-501f544888e2	dongdongvu2@gmail.com	Vũ Bá	Đông	lecturer	$2a$10$8AHPCsvA1HoazCr5E0XUVeciMH0aGs8X3.eSj6dxoAcHf0yV6w/g2	2025-03-26 21:56:24.828689+07	2025-03-26 21:56:24.828689+07	dongdongvu2	
c94554ed-3ed8-43fb-9cba-716c5f8ecea5	dongdongvu113@gmail.com		Đông	lecturer	$2a$10$WssvgP79D6C6vZuK/FhXkeakc5HItCd/Wz8Oljl/xW0p7qv9BGHdy	2025-03-27 12:05:50.109594+07	2025-03-27 12:05:50.109594+07	dongdongvu113	
2d536da8-fdf3-437b-a812-fb4e08aad955	Vubadong071102@gmail.com	Vũ Bá	Đông	lecturer	$2a$10$WssvgP79D6C6vZuK/FhXkeakc5HItCd/Wz8Oljl/xW0p7qv9BGHdy	2025-03-13 07:01:35.583166+07	2025-03-13 07:01:35.583166+07	vubadong071102	\N
62e33c55-0d38-4a0b-87cc-468038adf30e	student12@example.com	Last12	First12	student	hashedpassword12	2025-03-13 06:54:31.657573+07	2025-04-27 09:29:14.405312+07	\N	\N
f8455952-6f3f-485e-bd9b-942ce5bab472	student1@example.com	Trần nguyễn ánh	minh	student	hashedpassword1	2025-03-13 06:54:31.657573+07	2025-03-30 16:50:18.694081+07	\N	\N
37a947b9-9a4b-4c99-908f-20aeb3010785	student3@example.com	Vũ Đông	2	student	hashedpassword3	2025-03-13 06:54:31.657573+07	2025-03-30 16:50:32.590779+07	\N	\N
2e927240-2f96-4da4-9210-bc92b9ce7b8c	student5@example.com	Last5	First5	student	hashedpassword5	2025-03-13 06:54:31.657573+07	2025-03-30 17:00:05.059093+07	\N	\N
874b8d9b-f901-4063-8b15-4305a35f708e	student6@example.com	Last6	First6	student	hashedpassword6	2025-03-13 06:54:31.657573+07	2025-04-13 20:08:59.487427+07	\N	\N
0a7657d8-760e-40e1-8dcb-6ec8a56c8feb	2115239@example.com	Trần Văn	Nam	student	hashedpassword7	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
286c9234-c533-4ef6-8774-a5de92fb8452	2115253@example.com	Hồ Lê Anh	Quân	student	hashedpassword8	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
0ff33482-fc8b-4a12-b6e8-5ed58070d21e	2111881@example.com	Tăng Thế Ngọc	Song	student	hashedpassword29	2025-03-13 06:54:31.657573+07	2025-03-13 06:54:31.657573+07	\N	\N
3ef8aead-9239-43dc-9a7a-d9b3df374a99	student9@example.com	Last9	First9	student	hashedpassword9	2025-03-13 06:54:31.657573+07	2025-04-26 23:23:17.605803+07	\N	\N
fb15f4f8-4acc-43fa-80d2-5a6f0e2333f8	student10@example.com	Last10	First10	student	hashedpassword10	2025-03-13 06:54:31.657573+07	2025-04-27 08:56:37.299418+07	\N	\N
70c790b5-8ebd-4593-bf15-20bc404d408f	student11@example.com	Last11	First11	student	hashedpassword11	2025-03-13 06:54:31.657573+07	2025-04-27 09:18:59.869934+07	\N	\N
d64d1501-42f4-4685-b7fe-1f9785f4ece8	student4@example.com	First4	Last4	student	hashedpassword4	2025-03-13 06:54:31.657573+07	2025-04-27 09:31:11.353505+07	\N	\N
\.


--
-- TOC entry 3642 (class 2606 OID 17072)
-- Name: admins admins_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.admins
    ADD CONSTRAINT admins_pkey PRIMARY KEY (admin_id);


--
-- TOC entry 3633 (class 2606 OID 16855)
-- Name: attendance attendance_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.attendance
    ADD CONSTRAINT attendance_pkey PRIMARY KEY (attendance_id);


--
-- TOC entry 3617 (class 2606 OID 16789)
-- Name: cameras cameras_camera_ip_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.cameras
    ADD CONSTRAINT cameras_camera_ip_key UNIQUE (camera_url);


--
-- TOC entry 3619 (class 2606 OID 16787)
-- Name: cameras cameras_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.cameras
    ADD CONSTRAINT cameras_pkey PRIMARY KEY (camera_id);


--
-- TOC entry 3621 (class 2606 OID 16919)
-- Name: cameras cameras_socket_path_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.cameras
    ADD CONSTRAINT cameras_socket_path_key UNIQUE (socket_path);


--
-- TOC entry 3627 (class 2606 OID 16816)
-- Name: class_students class_students_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.class_students
    ADD CONSTRAINT class_students_pkey PRIMARY KEY (class_id, student_id);


--
-- TOC entry 3625 (class 2606 OID 16801)
-- Name: classes classes_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.classes
    ADD CONSTRAINT classes_pkey PRIMARY KEY (class_id);


--
-- TOC entry 3613 (class 2606 OID 16777)
-- Name: classrooms classrooms_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.classrooms
    ADD CONSTRAINT classrooms_pkey PRIMARY KEY (classroom_id);


--
-- TOC entry 3615 (class 2606 OID 16779)
-- Name: classrooms classrooms_room_name_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.classrooms
    ADD CONSTRAINT classrooms_room_name_key UNIQUE (room_name);


--
-- TOC entry 3611 (class 2606 OID 16763)
-- Name: courses courses_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.courses
    ADD CONSTRAINT courses_pkey PRIMARY KEY (course_id);


--
-- TOC entry 3605 (class 2606 OID 17160)
-- Name: lecturers lecturers_lecturer_code_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.lecturers
    ADD CONSTRAINT lecturers_lecturer_code_key UNIQUE (lecturer_code);


--
-- TOC entry 3607 (class 2606 OID 16749)
-- Name: lecturers lecturers_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.lecturers
    ADD CONSTRAINT lecturers_pkey PRIMARY KEY (lecturer_id);


--
-- TOC entry 3640 (class 2606 OID 17013)
-- Name: people_count_snapshots people_count_snapshots_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.people_count_snapshots
    ADD CONSTRAINT people_count_snapshots_pkey PRIMARY KEY (snapshot_id);


--
-- TOC entry 3631 (class 2606 OID 16835)
-- Name: schedules schedules_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.schedules
    ADD CONSTRAINT schedules_pkey PRIMARY KEY (schedule_id);


--
-- TOC entry 3646 (class 2606 OID 17178)
-- Name: semesters semester_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.semesters
    ADD CONSTRAINT semester_pkey PRIMARY KEY (semester_id);


--
-- TOC entry 3600 (class 2606 OID 16735)
-- Name: students students_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.students
    ADD CONSTRAINT students_pkey PRIMARY KEY (student_id);


--
-- TOC entry 3602 (class 2606 OID 17054)
-- Name: students students_student_code_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.students
    ADD CONSTRAINT students_student_code_key UNIQUE (student_code);


--
-- TOC entry 3644 (class 2606 OID 17074)
-- Name: admins uni_admins_admin_code; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.admins
    ADD CONSTRAINT uni_admins_admin_code UNIQUE (admin_code);


--
-- TOC entry 3609 (class 2606 OID 17081)
-- Name: lecturers uni_lecturers_lectainer_code; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.lecturers
    ADD CONSTRAINT uni_lecturers_lectainer_code UNIQUE (lectainer_code);


--
-- TOC entry 3638 (class 2606 OID 16936)
-- Name: attendance unique_schedule_student; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.attendance
    ADD CONSTRAINT unique_schedule_student UNIQUE (schedule_id, student_id);


--
-- TOC entry 3593 (class 2606 OID 16728)
-- Name: users users_email_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.users
    ADD CONSTRAINT users_email_key UNIQUE (email);


--
-- TOC entry 3595 (class 2606 OID 16726)
-- Name: users users_pkey; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.users
    ADD CONSTRAINT users_pkey PRIMARY KEY (user_id);


--
-- TOC entry 3597 (class 2606 OID 17031)
-- Name: users users_username_key; Type: CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.users
    ADD CONSTRAINT users_username_key UNIQUE (username);


--
-- TOC entry 3634 (class 1259 OID 16938)
-- Name: attendance_time_idx; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX attendance_time_idx ON public.attendance USING btree (attendance_time DESC);


--
-- TOC entry 3635 (class 1259 OID 16871)
-- Name: idx_attendance_student; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_attendance_student ON public.attendance USING btree (student_id, schedule_id);


--
-- TOC entry 3622 (class 1259 OID 16872)
-- Name: idx_cameras_classroom; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_cameras_classroom ON public.cameras USING btree (classroom_id);


--
-- TOC entry 3623 (class 1259 OID 16873)
-- Name: idx_cameras_ip; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_cameras_ip ON public.cameras USING btree (camera_url);


--
-- TOC entry 3628 (class 1259 OID 16869)
-- Name: idx_class_students; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_class_students ON public.class_students USING btree (student_id, class_id);


--
-- TOC entry 3603 (class 1259 OID 17161)
-- Name: idx_lecturers_code; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_lecturers_code ON public.lecturers USING btree (lecturer_code);


--
-- TOC entry 3629 (class 1259 OID 16870)
-- Name: idx_schedule_classroom; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_schedule_classroom ON public.schedules USING btree (class_id, classroom_id, start_time);


--
-- TOC entry 3598 (class 1259 OID 17055)
-- Name: idx_students_code; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_students_code ON public.students USING btree (student_code);


--
-- TOC entry 3591 (class 1259 OID 16866)
-- Name: idx_users_email; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX idx_users_email ON public.users USING btree (email);


--
-- TOC entry 3636 (class 1259 OID 16939)
-- Name: schedule_time_idx; Type: INDEX; Schema: public; Owner: postgres
--

CREATE INDEX schedule_time_idx ON public.attendance USING btree (schedule_id, attendance_time);


--
-- TOC entry 3664 (class 2620 OID 16913)
-- Name: classes validate_current_lessons; Type: TRIGGER; Schema: public; Owner: postgres
--

CREATE TRIGGER validate_current_lessons BEFORE INSERT OR UPDATE ON public.classes FOR EACH ROW EXECUTE FUNCTION public.check_current_lessons();


--
-- TOC entry 3659 (class 2606 OID 16856)
-- Name: attendance attendance_schedule_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.attendance
    ADD CONSTRAINT attendance_schedule_id_fkey FOREIGN KEY (schedule_id) REFERENCES public.schedules(schedule_id) ON DELETE CASCADE;


--
-- TOC entry 3660 (class 2606 OID 16861)
-- Name: attendance attendance_student_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.attendance
    ADD CONSTRAINT attendance_student_id_fkey FOREIGN KEY (student_id) REFERENCES public.students(student_id) ON DELETE CASCADE;


--
-- TOC entry 3654 (class 2606 OID 16790)
-- Name: cameras cameras_classroom_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.cameras
    ADD CONSTRAINT cameras_classroom_id_fkey FOREIGN KEY (classroom_id) REFERENCES public.classrooms(classroom_id) ON DELETE CASCADE;


--
-- TOC entry 3655 (class 2606 OID 16817)
-- Name: class_students class_students_class_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.class_students
    ADD CONSTRAINT class_students_class_id_fkey FOREIGN KEY (class_id) REFERENCES public.classes(class_id) ON DELETE CASCADE;


--
-- TOC entry 3656 (class 2606 OID 16822)
-- Name: class_students class_students_student_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.class_students
    ADD CONSTRAINT class_students_student_id_fkey FOREIGN KEY (student_id) REFERENCES public.students(student_id) ON DELETE CASCADE;


--
-- TOC entry 3651 (class 2606 OID 17090)
-- Name: courses courses_main_lecturer_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.courses
    ADD CONSTRAINT courses_main_lecturer_id_fkey FOREIGN KEY (main_lecturer_id) REFERENCES public.lecturers(lecturer_id) ON DELETE SET NULL;


--
-- TOC entry 3663 (class 2606 OID 17075)
-- Name: admins fk_admins_user; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.admins
    ADD CONSTRAINT fk_admins_user FOREIGN KEY (admin_id) REFERENCES public.users(user_id);


--
-- TOC entry 3661 (class 2606 OID 17019)
-- Name: people_count_snapshots fk_camera; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.people_count_snapshots
    ADD CONSTRAINT fk_camera FOREIGN KEY (camera_id) REFERENCES public.cameras(camera_id) ON DELETE CASCADE;


--
-- TOC entry 3652 (class 2606 OID 17147)
-- Name: courses fk_courses_main_lecturer; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.courses
    ADD CONSTRAINT fk_courses_main_lecturer FOREIGN KEY (main_lecturer_id) REFERENCES public.lecturers(lecturer_id);


--
-- TOC entry 3649 (class 2606 OID 17061)
-- Name: lecturers fk_lecturers_user; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.lecturers
    ADD CONSTRAINT fk_lecturers_user FOREIGN KEY (lecturer_id) REFERENCES public.users(user_id);


--
-- TOC entry 3662 (class 2606 OID 17014)
-- Name: people_count_snapshots fk_schedule; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.people_count_snapshots
    ADD CONSTRAINT fk_schedule FOREIGN KEY (schedule_id) REFERENCES public.schedules(schedule_id) ON DELETE CASCADE;


--
-- TOC entry 3653 (class 2606 OID 17179)
-- Name: courses fk_semester; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.courses
    ADD CONSTRAINT fk_semester FOREIGN KEY (semester_id) REFERENCES public.semesters(semester_id);


--
-- TOC entry 3647 (class 2606 OID 17056)
-- Name: students fk_students_user; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.students
    ADD CONSTRAINT fk_students_user FOREIGN KEY (student_id) REFERENCES public.users(user_id);


--
-- TOC entry 3650 (class 2606 OID 16752)
-- Name: lecturers lecturers_lecturer_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.lecturers
    ADD CONSTRAINT lecturers_lecturer_id_fkey FOREIGN KEY (lecturer_id) REFERENCES public.users(user_id) ON DELETE CASCADE;


--
-- TOC entry 3657 (class 2606 OID 16836)
-- Name: schedules schedules_class_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.schedules
    ADD CONSTRAINT schedules_class_id_fkey FOREIGN KEY (class_id) REFERENCES public.classes(class_id) ON DELETE CASCADE;


--
-- TOC entry 3658 (class 2606 OID 16841)
-- Name: schedules schedules_classroom_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.schedules
    ADD CONSTRAINT schedules_classroom_id_fkey FOREIGN KEY (classroom_id) REFERENCES public.classrooms(classroom_id) ON DELETE SET NULL;


--
-- TOC entry 3648 (class 2606 OID 16738)
-- Name: students students_student_id_fkey; Type: FK CONSTRAINT; Schema: public; Owner: postgres
--

ALTER TABLE ONLY public.students
    ADD CONSTRAINT students_student_id_fkey FOREIGN KEY (student_id) REFERENCES public.users(user_id) ON DELETE CASCADE;


-- Completed on 2025-04-27 15:37:19 +07

--
-- PostgreSQL database dump complete
--

