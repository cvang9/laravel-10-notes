##                Sending Notifications in laravel

Yeas, with this cool feature of laravel wé can send notifications to our users also.
notifications including variety of channels like - gmail, slack etc


create cmd- `php artisan make:notification NotificationName`
location- App\Notifications\NotificationName.php





{{


use Illuminate\Bus\Queueable;

class UserNotification extends Notification
{
    use Queueable;
    
    // Laravel notifications uses this queable trait by default why?
    // Because laravel send the notification asyncronously that means send mail in a related time not immediately because sending notifications is a quiet big task that consumes more time and if it will be executed syncronously then there may be a chance of execution blockage of our php script. 
    //  Now someone has to manage this asyncronous jobs in backend because php is focusing on executing the php script right now, so queue works this for us.



    //  Mention the platform which we wants to use : 'mail', 'database', 'broadcast', 'vonage' and 'slack'

    // mail -> Email
    // database -> Stores the content in a log file/table
    // broadcast -> Used in Web sockets when we need to broadcast a message
    // vonage, slack -> popular messaging apps

    public function via(object $notifiable): array
    {
        return ['mail'];
    }

    
}

}}






# Send Notifications 

We can send notifications in two ways:

1. Using Notifiable Trait
{{
 
namespace App\Models;
 
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;        // Trait imported
 
REMARK: Make sure you have included notifiable trait in your respective model.

class User extends Authenticatable
{
    use Notifiable;                           // Trait applied
}


`Now for each user instance a new method is available named` notify() ` that on called send the notification to the respective user`

use App\Notifications\NotificationName;
 
$user->notify(new NotificationName( $instance ));


}}


2. Using Facades

{{
    use Illuminate\Support\Facades\Notification;
    use App\Notifications\NotificationName;
    
    Notification::send( $users, new NotificationName() );
}}

STEP 1:

// Simply you can send nots without queue 
Go to .env and change the credentials


// Implementing Queue using database

-> Go To .env file and change 
`QUEUE_CONNECTION=database`

-> Make a Job
`php artisan make:job SendNewTicketNotification`

{{

// Job Structure
class SendNewTicketNotification implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $user;
    protected $ticket;

    public function __construct($user, $ticket)
    {
        $this->user = $user;
        $this->ticket = $ticket;
    }

    public function handle()
    {
        $this->user->notify(new NewTicketNotification($this->ticket));
    }
}


// Call a job from controller
$users = User::all();

// Create a new ticket
$ticket = Ticket::create([
    // Your ticket creation data here...
]);

// Dispatch the job for each user
foreach ($users as $user) {
    SendNewTicketNotification::dispatch($user, $ticket);
}

}}

-> Make a Job Table
`php artisan queue:table`

-> Run Queue
`php artisan queue:work`
