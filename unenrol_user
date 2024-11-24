<?php
define('CLI_SCRIPT', true);
require(__DIR__ . '/../../config.php');
require_once($CFG->dirroot . '/enrol/locallib.php');

global $DB;

// Find expired user enrollments.
$expired_enrolments = $DB->get_records_sql("
    SELECT ue.id, ue.userid, e.*
    FROM {user_enrolments} ue
    JOIN {enrol} e ON ue.enrolid = e.id
    WHERE ue.timeend > 0 AND ue.timeend < ?
", [time()]);

foreach ($expired_enrolments as $enrolment) {
    // Get the plugin type (e.g., 'manual', 'self', 'paystack').
    $enrolplugin = enrol_get_plugin($enrolment->enrol);
    
    if ($enrolplugin) {
        // Pass the full instance ($enrolment) to unenrol_user.
        $enrolplugin->unenrol_user((object)$enrolment, $enrolment->userid);
        echo "Unenrolled user with ID {$enrolment->userid} from enrolment method ID {$enrolment->id}\n";
    } else {
        echo "Could not find enrolment plugin for ID {$enrolment->id}\n";
    }
}
