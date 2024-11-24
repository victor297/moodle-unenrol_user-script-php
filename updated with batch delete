<?php
define('CLI_SCRIPT', true);
require(__DIR__ . '/../../config.php');
require_once($CFG->dirroot . '/enrol/locallib.php');

global $DB;

// Find all expired user enrollments.
$expired_enrolments = $DB->get_records_sql("
    SELECT ue.id AS userenrolid, ue.userid, e.id AS enrolid, e.courseid, e.enrol
    FROM {user_enrolments} ue
    JOIN {enrol} e ON ue.enrolid = e.id
    WHERE ue.timeend > 0 AND ue.timeend < ?
", [time()]);

if (!$expired_enrolments) {
    echo "No expired enrollments found.\n";
    exit(0);
}

echo "Found " . count($expired_enrolments) . " expired enrollments.\n";

// Process each expired enrollment.
foreach ($expired_enrolments as $enrolment) {
    // Retrieve the enrolment instance as an object.
    $enrol_instance = $DB->get_record('enrol', ['id' => $enrolment->enrolid]);

    if (!$enrol_instance) {
        echo "Error: Enrolment instance not found for enrolment ID {$enrolment->enrolid}\n";
        continue;
    }

    // Get the plugin type (e.g., 'manual', 'self', 'paystack').
    $enrolplugin = enrol_get_plugin($enrol_instance->enrol);

    if ($enrolplugin) {
        try {
            // Unenroll the user from the course.
            $enrolplugin->unenrol_user($enrol_instance, $enrolment->userid);
            echo "Unenrolled user with ID {$enrolment->userid} from enrolment ID {$enrol_instance->id}\n";
        } catch (Exception $e) {
            echo "Error unenrolling user with ID {$enrolment->userid}: " . $e->getMessage() . "\n";
        }
    } else {
        echo "Error: Enrolment plugin '{$enrol_instance->enrol}' not found for enrolment ID {$enrol_instance->id}\n";
    }
}
